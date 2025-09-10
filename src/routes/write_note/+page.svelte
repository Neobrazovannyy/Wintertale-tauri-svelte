<script lang="ts">
    import { onMount } from "svelte";
    let g_block_write_El: HTMLDivElement;
    let g_mini_console_El: HTMLInputElement;
    let g_focus_block: string = $state("block_write");

    let g_position_cursor_and_node: IndexTargetNodeAndWordsForConvert = $state({
        cursor_start: 0,
        cursor_end: 0,
        word_in_node_start: 0,
        word_in_node_end: 0,
        node_start : 0,
        node_end : 0,
        cursor_collapsed: true,
        obj_node_start: null,
        obj_node_end: null,
    });
    
    /*==================================================*/
    /*=== Interface ====================================*/

    interface IndexTargetNodeAndWordsForConvert{
        cursor_start: number;
        cursor_end: number;
        word_in_node_start: number;
        word_in_node_end: number;
        node_start: number;
        node_end: number;
        cursor_collapsed: boolean;
        obj_node_start: Node | null;
        obj_node_end: Node | null;
    }

    interface TargetNodeAndFlag{
        target_node: Node;
        flag: string;
    }

    //==============================================================================================
    //================================================================================ Load Page ===
    function LoadElementDOM(): void {
        g_block_write_El    = document.querySelector(".block_write")    as HTMLDivElement;
        g_mini_console_El   = document.querySelector(".mini_console")   as HTMLInputElement;

        g_block_write_El.addEventListener("click", ClickFieldBlockWrite);
        g_mini_console_El.addEventListener("keydown", EnteredCommandIntoMiniConsole);
    }
    onMount(LoadElementDOM);


    //===========================================================================================================
    //================================================================================ Hotkey & Walk Tree DOM === 
    
    // If you click on a field (block_write), it is in the focus variable (g_focus_block)
    function ClickFieldBlockWrite(): void {
        if(g_focus_block==="mini_console")
        {
            g_focus_block = "block_write";   //? document.activeElement
        }
    }
    
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
            HandleFuncConvert(ConvertLineTextToList, "find");
            SetCursorPosition();
        }
        else if(event.ctrlKey && event.key==="b"){
            event.preventDefault();
            event.stopPropagation();
            SetVariableCursorPosition();
            HandleFuncConvert(ConvertWordsAddTextDecoration, "b");
            SetCursorPosition();
        }
    }
    window.addEventListener("keydown", CheckHotkey);

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
                HandleFuncConvert(ConvertLineTextToHeader, wit_console_command[1]);
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
                HandleFuncConvert(ConvertLineTextToList, wit_console_command[1]);
            }
        }
        /*==================================*/
        /*============ New line ============*/
        /*==================================*/
        else if(wit_console_command.slice(0,2)==="nl"){
            /*----- dot, number -----*/
            if(wit_console_command.length <= 2){
                HandleFuncConvert(ConvertLineTextToAddNewLine, "1");
            }
            else{
                HandleFuncConvert(ConvertLineTextToAddNewLine, wit_console_command[2]);
            }
 
        }
        /*=======================================================*/
        /*============ Text decoration: bold, italic ============*/
        /*=======================================================*/
        else if(wit_console_command.slice(0,2)==="td"){
            // else if(wit_console_command[0]==="t"){
            /*----- bold -----*/
            if(["b", "i", "u"].includes(wit_console_command[2])){
                HandleFuncConvert(ConvertWordsAddTextDecoration, wit_console_command[2]);
            }
        }
        /*======================================*/
        /*============ Delete Style ============*/
        /*======================================*/
        else if(wit_console_command[0]==="/"){
            if(wit_console_command.length===1){
                HandleFuncConvert(ConvertLineTextDeleteStyle);
            }
        }

        SelectFocusElementSetVarTreeAndCurPos("block_write");
    }

    
    //========================================================================================================
    //================================================================================ Set Global Variable ===
    
    function SetTreeWalkerForBlockWrite(): TreeWalker{
        return document.createTreeWalker(
            g_block_write_El,
            NodeFilter.SHOW_ELEMENT,
            {
                acceptNode(node) {
                    if(node instanceof HTMLDivElement){
                        if(node.parentNode===g_block_write_El){
                            return NodeFilter.FILTER_ACCEPT;
                        }
                        return NodeFilter.FILTER_REJECT;
                    }
                    else{
                        return NodeFilter.FILTER_REJECT;
                    }
                }
            }
        );
    }

    // Finding and Set the value cursor position
    function SetVariableCursorPosition(): void {
        let tree_block_write: TreeWalker=SetTreeWalkerForBlockWrite();

        let val_node_start: Node | null=null;
        let val_node_end: Node | null=null;
        // Get all position
        let index_pos_cursor_start: number=0;
        let index_pos_cursor_end: number=0;
        let index_word_in_node_start: number=0;
        let index_word_in_node_end: number=0;
        let index_node_start: number=0;
        let index_node_end: number=0;
        
        // Get global position cursor
        let global_position_end: number=0;
        let global_position_start: number=0;
        let cursor_collapsed: boolean=true;


        let element_select: Selection = window.getSelection()!;
        let node_rang: Range=element_select.getRangeAt(0);
        let global_rang: Range=node_rang.cloneRange();
        global_rang.selectNodeContents(g_block_write_El);
        global_rang.setEnd(node_rang.endContainer, node_rang.endOffset);
        
        global_position_end=global_rang.toString().length;
        let text_node_start: string=node_rang.startContainer.textContent || "";
        let text_node_end: string=node_rang.endContainer.textContent || "";

        L("==========={Rang}===============");
        L(global_rang.toString());

        cursor_collapsed=node_rang.collapsed;
        if(!cursor_collapsed){
            // Calculate the global value of the start position of the cursor 
            global_rang.setStart(node_rang.startContainer, node_rang.startOffset);
            let select_text: string=global_rang.toString();
            let select_text_length: number=select_text.length;
            global_position_start=global_position_end-select_text_length;
        }
        else{
            global_position_start=global_position_end;
        }

        //<--- Set: index_node_start, index_node_end, index_pos_cursor_start, index_pos_cursor_end
        let count_symbol_global: number=0;
        let check_adding_index_node_start: boolean=true;
        let check_set_node_start: boolean=true;
        let current_node: Node;
        L("==========={Node walker}===============");
        while(current_node=tree_block_write?.nextNode() as Node){
            let node_content: string=current_node.textContent || "";
            let node_content_length: number=node_content.length;
            count_symbol_global+=node_content_length;
            L(current_node);
            L(node_content);
            // checking the starting node
            if(
                check_adding_index_node_start &&
                !cursor_collapsed &&
                count_symbol_global>=global_position_start &&
                node_content.slice(0,50)==text_node_start.slice(0,50)
            ){
                index_pos_cursor_start=node_content_length-(count_symbol_global-global_position_start);
                check_adding_index_node_start=false;
                val_node_start=current_node;
            }


            // checking the last node
            if(
                count_symbol_global>=global_position_end &&
                node_content.slice(0,50)==text_node_end.slice(0,50)
            ){
                index_pos_cursor_end=node_content_length-(count_symbol_global-global_position_end);
                val_node_end=current_node;
                break;
            }

            // adding one to the index of the starting node 
            if(!cursor_collapsed && check_adding_index_node_start) index_node_start++;
            // adding one to the index of the end node
            index_node_end++;
        }
        // I'm going back to the first node.
        if(tree_block_write) tree_block_write.currentNode = g_block_write_El;

        if(cursor_collapsed){
            index_pos_cursor_start=index_pos_cursor_end;
            index_node_start=index_node_end;
            val_node_start=val_node_end;
        }

        let list_words_in_node_start: string[] = text_node_start.split(" ").filter(x=>(x!=""));
        let list_words_in_node_end: string[] = text_node_end.split(" ").filter(x=>(x!=""));
        /*====== Check empty value ======*/
        //    "Myths are  ancient tales" => [ "Myths", "are", "", "ancient", "tales" ]
        //    +filter() =>  ["Myths", "are", "ancient", "tales"]

        //<--- Set: index_word_in_node_end
        count_symbol_global=0;
        for(var word_in_node of list_words_in_node_end){
            let word_in_node_length: number=word_in_node.length;
            count_symbol_global+=word_in_node_length;
            if(cursor_collapsed && count_symbol_global>=index_pos_cursor_end) break;
            if(!cursor_collapsed && count_symbol_global+1>=index_pos_cursor_end) break;
            count_symbol_global++;
            index_word_in_node_end++;
        }
        //<--- Set: index_word_in_node_start
        if(!cursor_collapsed){
            count_symbol_global=0;
            for(var word_in_node of list_words_in_node_start){
                let word_in_node_length: number=word_in_node.length;
                count_symbol_global+=word_in_node_length;
                if(count_symbol_global-1>=index_pos_cursor_start) break;
                count_symbol_global++;
                index_word_in_node_start++;
            }
        }
        else{
            index_word_in_node_start=index_word_in_node_end;
        }

        g_position_cursor_and_node={
            cursor_start: index_pos_cursor_start,
            cursor_end: index_pos_cursor_end,
            word_in_node_start: index_word_in_node_start,
            word_in_node_end: index_word_in_node_end,
            node_start: index_node_start,
            node_end: index_node_end,
            cursor_collapsed: cursor_collapsed,
            obj_node_start: val_node_start,
            obj_node_end: val_node_end,
        }
        L("==========={g_position_cursor_and_node}===============");
        L(index_node_end);
        L(index_word_in_node_end);
        L(index_pos_cursor_end);
    }


    function SetCursorPosition(): void {
        let index_target_node: number=g_position_cursor_and_node.node_end;
        let index_target_word_in_node: number=g_position_cursor_and_node.cursor_end;

        let tree_block_write: TreeWalker=SetTreeWalkerForBlockWrite();
        let current_node: Node | null;
        let count_index_node: number=0;

        while((current_node=tree_block_write?.nextNode())){
            if(count_index_node==index_target_node){
                let new_range: Range=document.createRange();
                new_range.collapse(true);
                new_range.setStart(current_node, index_target_word_in_node);

                let select_el = window.getSelection();
                if(select_el){
                    select_el.removeAllRanges();
                    select_el.addRange(new_range);
                }
                return;
            }
            count_index_node++;
        }
    }

    //------------------------------------------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Find Node in Tree DOM and Function Convert Call ---
    function HandleFuncConvert(CallFunc: (i_target: TargetNodeAndFlag)=> void, parameter_func: string = ""): void{
        let i_target: TargetNodeAndFlag={
            target_node: g_position_cursor_and_node.obj_node_end as Node,
            flag: parameter_func,
        }
        CallFunc(i_target);
    }
    
    
    //======================================================================================================
    //================================================================================ Convert Line Text ===

    //--------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- DEL ALL Style ---
    function ConvertLineTextDeleteStyle(i_target: TargetNodeAndFlag): void{
        let target_node: Node = i_target.target_node;

        let node_content = target_node?.textContent ?? "";
        let parent_element = target_node as HTMLDivElement;

        // delete: title
        parent_element.style.fontSize = "16px";
        parent_element.style.fontWeight = "300";
        // delete: list
        parent_element.style.padding = "0px";
        // delete: dot list
        if(node_content[0]==="•" && node_content[1]===" "){
            parent_element.textContent = parent_element.textContent?.slice(2) ?? "";
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

    //-------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Header ---
    function ConvertLineTextToHeader(i_target: TargetNodeAndFlag): void {
        let target_node: Node = i_target.target_node;
        let name_flag: string=i_target.flag;

        let parent_element: HTMLElement = target_node as HTMLDivElement;
        if(name_flag==="/"){
            parent_element.style.fontSize = "16px";
            parent_element.style.fontWeight = "300";
        }
        else if(name_flag==="1"){
            parent_element.style.fontSize = "40px";
            parent_element.style.fontWeight = "600";
        }
        else if(name_flag==="2"){
            parent_element.style.fontSize = "32px";
            parent_element.style.fontWeight = "600";
        }
        else if(name_flag==="3"){
            parent_element.style.fontSize = "24px";
            parent_element.style.fontWeight = "600";
        }
        else if(name_flag==="4"){
            parent_element.style.fontSize = "20px";
            parent_element.style.fontWeight = "600";
        }
        else if(name_flag==="5"){
            parent_element.style.fontSize = "16px";
            parent_element.style.fontWeight = "600";
        }
        else if(name_flag==="6"){
            parent_element.style.fontSize = "14px";
            parent_element.style.fontWeight = "600";
        }
    }

    //------------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- List: dot, number ---
    function ConvertLineTextToList(i_target: TargetNodeAndFlag): void {
        let target_node: Node = i_target.target_node;
        let name_flag: string=i_target.flag;

        let tree_block_write: TreeWalker=SetTreeWalkerForBlockWrite();
        
        // let parent_element: HTMLElement = target_node as HTMLElement;
        // if(parent_element){
        let parent_element: HTMLElement = target_node as HTMLElement;
        if(name_flag==="/")
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
        else if(name_flag==="d")
        {
            parent_element.style.paddingLeft = "20px";
            parent_element.textContent = "•" + " " + parent_element.textContent;
        }
        else if(name_flag==="n")
        {
            parent_element.style.paddingLeft = "20px";

            tree_block_write.currentNode = target_node;
            let previous_element_content: string = tree_block_write?.previousNode()?.textContent ?? "";
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
        else if(name_flag==="find")
        {
            tree_block_write.currentNode = target_node;
            let previous_element_content = tree_block_write?.previousNode()?.textContent ?? "";
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
        // }
    }

    //----------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Half new line ---
    function ConvertLineTextToAddNewLine(i_target: TargetNodeAndFlag): void {
        let parent_element: HTMLElement | null = i_target.target_node as HTMLDivElement;
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

    //-------------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Bold & Italic & Underline Font ---
    function ConvertWordsAddTextDecoration(i_target: TargetNodeAndFlag): void {
        if(i_target.flag==="b"){
            ConvertWordToBoldFont();
        }
    }

    //----------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Bold Font ---
    function ConvertWordToBoldFont(): void{
        let index_word_in_node_start: number=g_position_cursor_and_node.word_in_node_start;
        let index_word_in_node_end: number=g_position_cursor_and_node.word_in_node_end;
        let node_start: Node | null = g_position_cursor_and_node.obj_node_start;
        let node_end: Node | null = g_position_cursor_and_node.obj_node_end;

        // let fragment_convert_text: DocumentFragment = document.createDocumentFragment();
        let fragment_convert_text: HTMLDivElement = document.createElement("div");
        let list_words_in_node: string[]=node_start?.textContent?.split(" ") ?? [""];

        let count_index_word: number=0;
        if(node_start==node_end)
        {
            let list_words_length=list_words_in_node.length
            for(var index_word_node=0; index_word_node<list_words_length; index_word_node++)
            {
                if(index_word_node>=index_word_in_node_start && index_word_node<=index_word_in_node_end && list_words_in_node[index_word_node]!="")
                {
                    let el_word_convert: HTMLSpanElement=document.createElement("span");
                    el_word_convert.textContent=list_words_in_node[index_word_node];
                    el_word_convert.style.fontWeight="600";
                    el_word_convert.className="wtr_word_textDecoration_bold";

                    fragment_convert_text.appendChild(el_word_convert);
                    if(index_word_node!=list_words_length-1){
                        fragment_convert_text.appendChild(document.createTextNode(" "));
                    }
                }
                else{
                    if(list_words_in_node[index_word_node]!=""){
                        fragment_convert_text.appendChild(document.createTextNode(list_words_in_node[index_word_node]));
                    }
                    else{
                        fragment_convert_text.appendChild(document.createTextNode(" "));
                    }

                    
                    if(index_word_node!=list_words_length-1){
                        fragment_convert_text.appendChild(document.createTextNode(" "));
                    }
                }
            }
            L("===========================>");
            L(node_start?.parentElement);
            L("<===========================");
            node_start?.parentElement?.replaceWith(fragment_convert_text);
            node_start?.parentElement?.normalize();
            // node_start?.parentElement?.replaceChild(fragment_convert_text, node_start);
        }

    }

    function ConvertWordsToItalicFont(current_node: Node): void{

    }

    
    //=========================================================================================================================
    //================================================================================ Helper functions (can be safely removed)
    
    function L(variable: unknown): void {
        console.log(variable);
    }

    function LV(description: string="", variable: unknown=""): void {
        console.log(`${description}: ${variable}`);
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

        <div>Myths are ancient, timeless tales,</div><div><br></div><div>Of gods and heroes, monsters, and whales.</div><div><br></div><div>They tried to explain the world's creation,</div><div>And <span style="font-weight: 600">teach</span> a les-son to every nation.</div><div>More than just stories from long ago,</div><div>They show us truths that we all know.</div>
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
