<script lang="ts">
    // import type { ChannelNameHwb } from "sass";
    import { onMount } from "svelte";
    import { slide } from "svelte/transition";
    let g_block_write_El: HTMLDivElement;
    let g_mini_console_El: HTMLInputElement;
    let g_focus_block: string = $state("block_write");

    let g_tree_block_write: TreeWalker | null = $state(null);

    let g_cursor_pos_symbol: PositionStartLine = $state({
        start_position: 0,
        position: 0,
        start: false,
        collapsed: true,
        amount_words: 1,
        empty_node: false,
    });
    
    /*==================================================*/
    /*=== Interface ====================================*/

    interface TargetNodeForConvert {
        node: Node;
        flag: string;
    }
    
    interface TargetNodeAndWordsForConvert {
        start_node: Node;
        index_start_word_in_node: number;
        amount_words: number;
        flag: string;
    }

    interface PositionStartLine {
        start_position: number;
        position: number;
        start: boolean;
        collapsed: boolean;
        amount_words: number;
        empty_node: boolean;
    }
    
    //==============================================================================================
    //================================================================================ Load Page ===
    function LoadElementDOM(): void {
        g_block_write_El    = document.querySelector(".block_write")    as HTMLDivElement;
        g_mini_console_El   = document.querySelector(".mini_console")   as HTMLInputElement;

        g_block_write_El.addEventListener("click", ClickFieldBlockWrite);
        g_mini_console_El.addEventListener("keydown", EnteredCommandIntoMiniConsole);

        // SetVariableTreeWalker();
    }
    onMount(LoadElementDOM);

    
    //========================================================================================================
    //================================================================================ Set Global Variable ===

    // Create TreeWalker
    function SetVariableTreeWalker(element_focus: Element = g_block_write_El): void {
        g_tree_block_write = document.createTreeWalker(
            element_focus,
            NodeFilter.SHOW_ELEMENT | NodeFilter.SHOW_TEXT,
            {
                acceptNode(node) {
                    if(node instanceof HTMLDivElement){
                        return NodeFilter.FILTER_SKIP;
                    }
                    else{
                        return NodeFilter.FILTER_ACCEPT;
                    }
                }
            }
        );
    }
    
    // Finding and Set the value cursor position
    function SetVariableCursorPosition(): void {        
        let position_start: number;
        let position_end: number;
        let check_start_line: boolean;
        let cursor_collapsed: boolean;
        let num_words_in_select_text: number=1;
        let check_empty_node: boolean;

        let select_el: Selection = window.getSelection()!;
        let range_line: Range = select_el.getRangeAt(0);
        let rang_line_end_cont: Node = range_line.endContainer;
        let rang_line_start_cont: Node = range_line.startContainer;
        let rang_line_start_offset: number = range_line.startOffset;
        let rang_line_end_offset: number = range_line.endOffset;

        let range_global: Range = range_line.cloneRange();
        range_global.selectNodeContents(g_block_write_El);

        //---> g_cursor_pos_symbol.position
        range_global.setEnd(rang_line_end_cont, rang_line_end_offset);
        position_end=range_global.toString().length;
        position_start=position_end

        //---> g_cursor_pos_symbol.start
        check_start_line = (rang_line_end_offset==0) ? (true) : (false);
              
        //---> g_cursor_pos_symbol.empty_node
        check_empty_node = (rang_line_end_cont.nodeValue==null) ? (true) : (false);

        //---> g_cursor_pos_symbol.collapsed
        cursor_collapsed=range_line.collapsed;
        //---> g_cursor_pos_symbol.start_position
        if(!cursor_collapsed){
            range_global.setStart(rang_line_start_cont, rang_line_start_offset);
            let rang_glob_str: string = range_global.toString();
            position_start-=rang_glob_str.length;
            num_words_in_select_text=rang_glob_str.trim().split(" ").length;
        }
    
        
        g_cursor_pos_symbol={
            start_position: position_start,
            position: position_end,
            start: check_start_line,
            collapsed: cursor_collapsed,
            amount_words: num_words_in_select_text,
            empty_node: check_empty_node
        }

        // L("-------------------------------------------->");
        // L(g_cursor_pos_symbol.start_position);
        // L(g_cursor_pos_symbol.position);
        // L(g_cursor_pos_symbol.start);
        // L(g_cursor_pos_symbol.collapsed);
        // L(g_cursor_pos_symbol.amount_words);
        // L(g_cursor_pos_symbol.empty_node);
        // L("<--------------------------------------------");
    }

    // Setting the cursor position
    function SetCursorPosition(): void {
        SetVariableTreeWalker();

        let count_symbol: number = 0;
        let current_node: Node | null;

        while(g_tree_block_write?.nextNode())
        {
            current_node = g_tree_block_write.currentNode ?? null;
            let length_str_current_node = current_node.nodeValue?.length ?? 0;

            if(count_symbol+length_str_current_node >= g_cursor_pos_symbol.position)
            {
                let position_on_line: number = 0;
                let new_range: Range = document.createRange();
                new_range.collapse(true);
                if(g_cursor_pos_symbol.start && g_cursor_pos_symbol.position!=0)
                {
                    g_tree_block_write?.nextNode();
                    current_node = g_tree_block_write.currentNode ?? null;
                    position_on_line=0;
                }
                else{
                    position_on_line=g_cursor_pos_symbol.position-count_symbol;
                }

                // Checking the compliance of the lines
                if(g_cursor_pos_symbol.empty_node != (current_node.nodeValue==null))
                {
                    g_tree_block_write?.nextNode();
                    current_node = g_tree_block_write.currentNode ?? null;
                }

                new_range.setStart(current_node, position_on_line);
                

                let select_el = window.getSelection();
                if(select_el){
                    select_el.removeAllRanges();
                    select_el.addRange(new_range);
                }

                g_tree_block_write.currentNode = g_block_write_El;
                return;
            }
            count_symbol+=length_str_current_node;
        }
    }


    //===========================================================================================================
    //================================================================================ Hotkey & Walk Tree DOM ===
      
    // If you click on a field (block_write), it is in the focus variable (g_focus_block)
    function ClickFieldBlockWrite(): void {
        if(g_focus_block==="mini_console")
        {
            g_focus_block = "block_write";   //? document.activeElement
        }
    }
    
    function CheckHotkey(event: KeyboardEvent) {
        // Key down: CTRL + SHIFT;
        // if(event.ctrlKey && event.shiftKey)
        if(event.code === "Tab")
        {
            event.preventDefault();
            event.stopPropagation();

            // focus moves to the object: "mini console"
            if(g_focus_block==="block_write")
            {
                SelectFocusElementSetVarTreeAndCurPos("mini_console");
            }
            // focus moves to the object: "block write"
            else if(g_focus_block==="mini_console")
            {
                SelectFocusElementSetVarTreeAndCurPos("block_write");
            }
        }
        else if(event.ctrlKey && event.key==="d"){
            event.preventDefault();
            event.stopPropagation();

            SetVariableCursorPosition();
            FindAndConvertLineInTreeDOM(ConvertLineTextToList, "find");
            SetCursorPosition();
        }
        else if(event.ctrlKey && event.key==="b"){
            event.preventDefault();
            event.stopPropagation();
            SetVariableCursorPosition();
            FindAndConvertWordsInTreeDOM(ConvertWordsAddTextDecoration, "b");
            SetCursorPosition();
        }
    }
    window.addEventListener("keydown", CheckHotkey);


    //------------------------------------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Select Focus && CALL: Var Cursor Position ---
    function SelectFocusElementSetVarTreeAndCurPos(focus_element: string): void {
        if(focus_element==="block_write"){
            g_block_write_El.focus();
            SetCursorPosition();
            g_mini_console_El.value="";
            g_focus_block="block_write";
        }
        else if(focus_element==="mini_console"){
            SetVariableCursorPosition();
            g_mini_console_El.focus();
            g_focus_block = "mini_console";
        }
    }


    //-----------------------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Enter Text Into Mini Console ---
    function EnteredCommandIntoMiniConsole(event: KeyboardEvent): void {
        if(event.code!=="Enter") return;
        
        event.preventDefault();
        event.stopPropagation();

        let wit_console_command: string = g_mini_console_El.value;

        /*==============================================*/
        /*============ Header of N-th order ============*/
        /*==============================================*/
        if(wit_console_command[0]==="h"){
            /*----- <h1>...<h6> -----*/
            if(
                wit_console_command.length === 2 &&
                ["/", "1","2","3","4","5","6"].includes(wit_console_command[1])
            ){
                FindAndConvertLineInTreeDOM(ConvertLineTextToHeader, wit_console_command[1]);
            }
        }
        /*==============================*/
        /*============ List ============*/
        /*==============================*/
        else if(wit_console_command[0]==="l"){
            /*----- dot, number -----*/
            if(
                wit_console_command.length === 2 &&
                ["/", "d", "n"].includes(wit_console_command[1])
            ){
                FindAndConvertLineInTreeDOM(ConvertLineTextToList, wit_console_command[1]);
            }
        }
        /*==================================*/
        /*============ New line ============*/
        /*==================================*/
        else if(wit_console_command.slice(0,2)==="nl"){
            /*----- dot, number -----*/
            if(wit_console_command.length <= 2){
                FindAndConvertLineInTreeDOM(ConvertLineTextToAddNewLine, "1");
            }
            else{
                FindAndConvertLineInTreeDOM(ConvertLineTextToAddNewLine, wit_console_command[2]);
            }
 
        }
        /*=======================================================*/
        /*============ Text decoration: bold, italic ============*/
        /*=======================================================*/
        else if(wit_console_command.slice(0,2)==="td"){
            // else if(wit_console_command[0]==="t"){
            /*----- dot, number -----*/
            if(wit_console_command[2]==="b"){
                FindAndConvertWordsInTreeDOM(ConvertWordsAddTextDecoration, wit_console_command[2]);
            }
            else{
                // FindAndConvertWordsInTreeDOM(ConvertLineTextToAddNewLine, wit_console_command[2]);
            }
 
        }
        /*======================================*/
        /*============ Delete Style ============*/
        /*======================================*/
        else if(wit_console_command[0]==="/"){
            if(wit_console_command.length===1){
                FindAndConvertLineInTreeDOM(ConvertLineTextDeleteStyle);
            }
        }

        SelectFocusElementSetVarTreeAndCurPos("block_write");
    }


    //--------------------------------------------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Find Line in TreeWalker and Function Convert Call ---
    function FindAndConvertLineInTreeDOM(CallFunc: (i_target: TargetNodeForConvert)=> void, parameter_func: string = ""): void {
        SetVariableTreeWalker();

        let count_symbol: number = 0;
        let current_node: Node | null;
        let i_target: TargetNodeForConvert;

        while(g_tree_block_write?.nextNode())
        {
            current_node = g_tree_block_write.currentNode ?? null;
            let length_str_current_node = current_node.nodeValue?.length ?? 0;

            if(count_symbol+length_str_current_node >= g_cursor_pos_symbol.position)
            {
                let new_range: Range = document.createRange();
                if(g_cursor_pos_symbol.start && g_cursor_pos_symbol.position!=0)
				{
                    g_tree_block_write?.nextNode();
                    current_node = g_tree_block_write.currentNode ?? null;
				}

				// Checking the compliance of the lines
				if(g_cursor_pos_symbol.empty_node != (current_node.nodeValue==null))
				{
                    g_tree_block_write?.nextNode();
                    current_node = g_tree_block_write.currentNode ?? null;
				}


				i_target={
                    node: current_node,
                    flag: parameter_func
				}
				/*-----------------------------------*/
				CallFunc(i_target);
				/*-----------------------------------*/

				let select_el = window.getSelection();
				if(select_el){
                    select_el.removeAllRanges();
                    select_el.addRange(new_range);
				}

				g_tree_block_write.currentNode = g_block_write_El;
				return;
            }
            count_symbol+=length_str_current_node;
        }
    }

    //---------------------------------------------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Find Words in TreeWalker and Function Convert Call ---
    function FindAndConvertWordsInTreeDOM(CallFunc: (i_target: TargetNodeAndWordsForConvert)=>void, parameter_func: string=""): void {
        SetVariableTreeWalker();

        let count_symbol_in_each_line: number = 0;
        let current_node: Node | null;
        let current_node_str: string;
        let i_target: TargetNodeAndWordsForConvert;

        while(g_tree_block_write?.nextNode())
        {
            current_node = g_tree_block_write.currentNode ?? null;
            current_node_str = current_node.nodeValue ?? "";
            let length_str_current_node = current_node_str.length ?? 0;

            if(count_symbol_in_each_line+length_str_current_node >= g_cursor_pos_symbol.start_position)
            {
                let index_cursor_in_line: number = g_cursor_pos_symbol.start_position-count_symbol_in_each_line;
                let new_range: Range = document.createRange();

                if(g_cursor_pos_symbol.start && g_cursor_pos_symbol.start_position!=0)
                {
                    g_tree_block_write?.nextNode();
                    current_node = g_tree_block_write.currentNode ?? null;
                }

                // Checking the compliance of the lines
                if(g_cursor_pos_symbol.empty_node != (current_node.nodeValue==null))
                {
                    g_tree_block_write?.nextNode();
                    current_node = g_tree_block_write.currentNode ?? null;
                }

                //<--- The target node is correctly selected: g_tree_block_write.currentNode

                // I'm looking for the index of the first word
                let list_words_in_line_current_node: string[] = current_node_str.split(" ");
                let index_start_word: number=0;
                let count_symbols_in_words: number=0;
                for(var word_from_current_node of list_words_in_line_current_node)
                {
                    count_symbols_in_words+=word_from_current_node.length-1;
                    if(count_symbols_in_words>=index_cursor_in_line){
                        break;
                    }
                    else{
                        index_start_word++;
                    }
                    count_symbols_in_words++; // I'm adding the value of one space before the word
                }

				i_target={
                    start_node: current_node,
                    index_start_word_in_node: index_start_word,
                    amount_words: g_cursor_pos_symbol.amount_words,
                    flag: parameter_func
				}
				/*-----------------------------------*/
				CallFunc(i_target);
				/*-----------------------------------*/

				let select_el = window.getSelection();
				if(select_el){
                    select_el.removeAllRanges();
                    select_el.addRange(new_range);
				}

				g_tree_block_write.currentNode = g_block_write_El;
				return;
            }
            count_symbol_in_each_line+=length_str_current_node;
        }
    }

    //======================================================================================================
    //================================================================================ Convert Line Text ===

    //--------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- DEL ALL Style ---
    function ConvertLineTextDeleteStyle(i_target: TargetNodeForConvert): void{
        if(i_target.node.parentElement){
            let node_content = i_target.node?.textContent ?? "";
            let parent_element = i_target.node.parentElement;

            // delete: title
            parent_element.style.fontSize = "16px";
            parent_element.style.fontWeight = "300";
            // delete: list
            parent_element.style.padding = "0px";
            // delete: dot list
            if(node_content[0]==="•" && node_content[1]===" "){
                parent_element.textContent = parent_element.textContent.slice(2);
            }
            // delete: number list
            else if(["0","1","2","3","4","5","6","7","8","9"].includes(node_content[0]))
            {
                let num_pos_bracket_node: number = node_content.indexOf(")");
                let serial_number_node: number = Number(node_content.slice(0,num_pos_bracket_node));
                if(num_pos_bracket_node!==-1 && !isNaN(serial_number_node))
                {
                    parent_element.textContent = node_content.slice(num_pos_bracket_node+2);
                }
            }
        }
    }


    //-------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Header ---
    function ConvertLineTextToHeader(i_target: TargetNodeForConvert): void {
        if(i_target.node.parentElement){
            let parent_element: HTMLElement = i_target.node.parentElement;
            if(i_target.flag==="/"){
                parent_element.style.fontSize = "16px";
                parent_element.style.fontWeight = "300";
            }
            else if(i_target.flag==="1"){
                parent_element.style.fontSize = "40px";
                parent_element.style.fontWeight = "600";
            }
            else if(i_target.flag==="2"){
                parent_element.style.fontSize = "32px";
                parent_element.style.fontWeight = "600";
            }
            else if(i_target.flag==="3"){
                parent_element.style.fontSize = "24px";
                parent_element.style.fontWeight = "600";
            }
            else if(i_target.flag==="4"){
                parent_element.style.fontSize = "20px";
                parent_element.style.fontWeight = "600";
            }
            else if(i_target.flag==="5"){
                parent_element.style.fontSize = "16px";
                parent_element.style.fontWeight = "600";
            }
            else if(i_target.flag==="6"){
                parent_element.style.fontSize = "14px";
                parent_element.style.fontWeight = "600";
            }
        }
    }


    //------------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- List: dot, number ---
    function ConvertLineTextToList(i_target: TargetNodeForConvert): void {
        let parent_element: HTMLElement | null = i_target.node.parentElement;
        if(parent_element){
            let parent_element = i_target.node.parentElement;
            if(i_target.flag==="/")
            {
                let parent_element_content = parent_element.textContent ?? "";
                parent_element.style.paddingLeft = "0px";
                
                if(parent_element_content[0]==="•")
                {
                    if(parent_element_content[1]===" "){
                        parent_element.textContent = parent_element_content.slice(2);
                    }
                    else{
                        parent_element.textContent = parent_element_content.slice(1);
                    }
                }
                else if(["0","1","2","3","4","5","6","7","8","9"].includes(parent_element_content[0]))
                {
                    let num_pos_bracket: number = parent_element_content.indexOf(")");
                    let serial_number: number = Number(parent_element_content.slice(0,num_pos_bracket));
                    if(!isNaN(serial_number) && parent_element_content[num_pos_bracket+1]===" "){
                        parent_element.textContent = parent_element_content.slice(num_pos_bracket+2);
                    }
                }
            }
            else if(i_target.flag==="d")
            {
                parent_element.style.paddingLeft = "20px";
                parent_element.textContent = "•" + " " + parent_element.textContent;
            }
            else if(i_target.flag==="n")
            {
                parent_element.style.paddingLeft = "20px";

                let previous_element_content: string = g_tree_block_write?.previousNode()?.textContent ?? "";
                let num_pos_bracket_previous: number = previous_element_content.indexOf(")");
                if(num_pos_bracket_previous===-1){
                    parent_element.textContent = `1) ${parent_element.textContent}`;
                }
                else{
                    let serial_number_parent: number = Number(previous_element_content.slice(0,num_pos_bracket_previous));
                    if(!isNaN(serial_number_parent))
                    {
                        let parent_element_content = parent_element.textContent;
                        let num_pos_bracket: number = parent_element_content.indexOf(")");
                        let serial_number: number = Number(parent_element_content.slice(0,num_pos_bracket));
                        if(num_pos_bracket!==-1 && !isNaN(serial_number)){
                            parent_element.textContent = `${serial_number_parent+1}) ${parent_element_content.slice(num_pos_bracket+2)}`;
                        }
                        else{
                            parent_element.textContent = `${serial_number_parent+1}) ${parent_element_content}`;
                        }
                    }
                }
            }
            else if(i_target.flag==="find")
            {
                let previous_element_content = g_tree_block_write?.previousNode()?.textContent ?? "";
                if(previous_element_content[0]==="•")
                {
                    parent_element.style.paddingLeft = "20px";
                    parent_element.textContent = "•" + " " + parent_element.textContent;
                }
                else if(["0","1","2","3","4","5","6","7","8","9"].includes(previous_element_content[0]))
                {
                    parent_element.style.paddingLeft = "20px";

                    let num_pos_bracket_previous: number = previous_element_content.indexOf(")");
                    if(num_pos_bracket_previous===-1){
                        parent_element.textContent = `1) ${parent_element.textContent}`;
                    }
                    else{
                        let serial_number_parent: number = Number(previous_element_content.slice(0,num_pos_bracket_previous));
                        if(!isNaN(serial_number_parent))
                        {
                            let parent_element_content = parent_element.textContent;
                            let num_pos_bracket: number = parent_element_content.indexOf(")");
                            let serial_number: number = Number(parent_element_content.slice(0,num_pos_bracket));
                            if(num_pos_bracket!==-1 && !isNaN(serial_number)){
                                parent_element.textContent = `${serial_number_parent+1}) ${parent_element_content.slice(num_pos_bracket+2)}`;
                            }
                            else{
                                parent_element.textContent = `${serial_number_parent+1}) ${parent_element_content}`;
                            }
                        }
                    }
                }
            }
        }
    }


    //----------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Half new line ---
    function ConvertLineTextToAddNewLine(i_target: TargetNodeForConvert): void {
        let parent_element: HTMLElement | null = i_target.node.parentElement;
        if(parent_element===null) return;

        if(i_target.flag=="/"){
            parent_element.style.paddingBottom = `0px`;
        }

        let order_new_line_number: number | string = Number(i_target.flag);
        if(!isNaN(order_new_line_number)){
            parent_element.style.paddingBottom = `${4*order_new_line_number}px`;
        }
    }


    //=======================================================================================================
    //================================================================================ Convert Words Text ===

    //---------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Bold & Italic Font
    function ConvertWordsAddTextDecoration(i_target: TargetNodeAndWordsForConvert): void {
        if(i_target.flag==="b"){
            ConvertWordsToBoldFont(i_target);
        }
    }

    function ConvertWordsToBoldFont(i_target: TargetNodeAndWordsForConvert): void{
        let start_node: Node = i_target.start_node;
        let count_words: number=0;
        let first_node_fragment: DocumentFragment = document.createDocumentFragment();

        let list_words_in_current_node: string[] = i_target.start_node.nodeValue?.trim().split(" ") || [""];
        let list_words_len: number=list_words_in_current_node.length;
        for(var index_word=0; index_word<list_words_len; index_word++)
        {
            let target_word: string=list_words_in_current_node[index_word];

            // create an element of type (text)
            if(index_word<i_target.index_start_word_in_node || count_words>=i_target.amount_words){
                let word_not_convert: Text;
                if(index_word==list_words_len-1){
                    word_not_convert = document.createTextNode(target_word);
                }
                else{
                    word_not_convert = document.createTextNode(target_word+" ");
                }
                first_node_fragment.appendChild(word_not_convert);
                continue;
            };

            // create an element (span) for the word and convert it
            let el_word_convert: HTMLSpanElement = document.createElement("span");
            el_word_convert.textContent=target_word;
            el_word_convert.style.fontWeight="600";
            el_word_convert.className="wtr_textDecoration_bold";

            first_node_fragment.appendChild(el_word_convert);
            if(index_word!=list_words_len){
                first_node_fragment.appendChild(document.createTextNode(" "));
            }

            count_words++;
        }
        start_node.parentElement?.replaceChild(first_node_fragment, start_node);

        if(count_words>=i_target.amount_words) return;


        // while(g_tree_block_write?.nextNode())
        // {
        //     L(g_tree_block_write.currentNode.nodeValue);
        // }
    }

    
    //=========================================================================================================================
    //================================================================================ Helper functions (can be safely removed)
    
    function L(variable: unknown): void {
        console.log(variable);
    }

    function LV(description: string="", variable: unknown=""): void {
        console.log(`${description}: ${variable}`);
    }

    // "&#149;"
    function ConvertToHTML(html_string: string): string {
        const doc: Document = new DOMParser().parseFromString(html_string, 'text/html');
        return doc.documentElement.textContent || "";
    }

</script>


<!-- ! on:paste={when input data}, you need to check the data for special characters! -->
<div class="record_field">

    <div
        class="block_write"
        contenteditable="true" 
        autofocus
    >
        <!-- <div>
            I welcome you to a winter&rsquo;s fairy tale &#10052;
        </div> -->

        <!-- <div>Title 1</div><div>pop</div><div>kik</div><div>xcx</div><div><br></div><div>Title 2</div><div>loli</div><div>:)</div><div><br></div><div>Title 3</div><div>up &amp; down</div><div><br></div><div>@End</div> -->

        <div>Myths are ancient, timeless tales,</div><div><br></div><div>Of gods and heroes, monsters, and whales.</div><div><br></div><div>They tried to explain the world's creation,</div><div>And teach a lesson to every nation.</div><div>More than just stories from long ago,</div><div>They show us truths that we all know.</div>
    </div>

    <input
        class="mini_console"
        type="text"
        placeholder="TAB"
    >

</div>


<style lang="scss">
    /*==font==*/
    $absolute_font_family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    $absolute_font_weight: 300;
    /*==color==*/
        // text
    $text_white: rgb(229, 240, 255);
    $text_and_border_lite_blue: #5b7bac;
        // border
    // $border_lite_blue: #5C7198;
        // background
    $bg_dark_grey: #161b24;         // first choice
    $bg_more_dark_grey: #141820;    // first choice (more dark)
    $bg_dark_blue: #172030;         // second choice

    *{ //del
        // border: 2px solid #000;
    }

    .record_field{
        display: flex;
        flex-direction: column;
        align-items: center;
        width: 100%;
        height: 100%;
    }

    /*===================*/
    /*=== Block Write ===*/
    /*===================*/

    .block_write{
        color: $text_white;
        font-size: 16px;
        letter-spacing: 1.6px;
        font-family: $absolute_font_family;
        font-weight: $absolute_font_weight;
        background-color: $bg_more_dark_grey;
        width: 794px;
        height: 100%;
        padding: 30px 50px;
        margin: 5px;
        border: 0px;
        border-radius: 5px;
        /*-- Word Transfer --*/
        overflow-y: auto;
        overflow-x: hidden;
        white-space: pre-wrap;  /* transfer short word  */
        word-wrap: break-word;  /* transferring long words */
        /**/
        scrollbar-width: thin;
        scrollbar-gutter: stable;
        resize: none;

        &:focus{
            color: $text_white;
        }
    }

    .block_write:focus{
        color: $text_white;
        outline: none;
    }
    
    .block_write::-webkit-scrollbar {
        width: 5px;
        height: 10px;
        
        &:hover{
            width: 10px;
        }
    }

    .block_write::-webkit-scrollbar-track {
        background: #00000000;
    }

    .block_write::-webkit-scrollbar-thumb {
        background-color: rgba(229, 240, 255, 0.2);
        border-radius: 3px;
        border: 0px;

        &:hover{
            background-color: rgba(229, 240, 255, 0.6);
        }
    }
    
    .mini_console{
        position: absolute;
        background-color: $bg_more_dark_grey;
        color: $text_white;
        right: 0;
        bottom: 25px;
        height: 20px;
        border: 2px solid $text_and_border_lite_blue;
        pointer-events: none;

        &:focus{
            background-color: $text_and_border_lite_blue;
            outline: none;
            box-shadow: none;
        }
    }
</style>
