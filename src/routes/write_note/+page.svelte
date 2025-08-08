<script lang="ts">
    import { onMount } from "svelte";
    let g_block_write_El: HTMLDivElement;
    let g_mini_console_El: HTMLInputElement;
    let g_focus_block: string = $state("block_write");

    let g_tree_block_write: TreeWalker | null = $state(null);

    interface PositionStartLine {
        position: number;
        start: boolean;
    }
    let g_cursor_pos_symbol: PositionStartLine = $state({
        position: 0,
        start: false
    });


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
            // NodeFilter.SHOW_ELEMENT | NodeFilter.SHOW_TEXT,
            NodeFilter.SHOW_TEXT,
            null
        );
    }
    
    // Finding and Set the value cursor position
    function SetVariableCursorPosition(): void {
        let select_el: Selection = window.getSelection()!;
        let range_line: Range = select_el.getRangeAt(0);

        let range_global: Range = range_line.cloneRange();
        range_global.selectNodeContents(g_block_write_El);  //select all text
        range_global.setEnd(range_line.endContainer, range_line.endOffset);

        g_cursor_pos_symbol={
            position: range_global.toString().length,
            start: range_line.endOffset==0 ? true : false
        }
    }

    // Setting the cursor position
    function SetCursorPosition(): void {
        SetVariableTreeWalker();

        let count_symbol: number = 0;
        let current_node: Node | null;

        while(g_tree_block_write?.nextNode())
        {
            current_node = g_tree_block_write.currentNode ?? null;
            if(current_node.nodeType===Node.TEXT_NODE)
            {
                let length_str_current_node = current_node.nodeValue?.length ?? 0;

                if(count_symbol+length_str_current_node >= g_cursor_pos_symbol.position)
                {
                    let position_on_line: number = 0;
                    let new_range: Range = document.createRange();
                    new_range.collapse(true);
                    if(g_cursor_pos_symbol.start)
                    {
                        g_tree_block_write?.nextNode();
                        current_node = g_tree_block_write.currentNode ?? null;
                        position_on_line=0;
                    }
                    else{
                        position_on_line=g_cursor_pos_symbol.position-count_symbol;
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
        if(event.ctrlKey && event.shiftKey)
        {
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
        else if(event.ctrlKey && event.key==="d")
        {
            event.preventDefault();
            event.stopPropagation();

            SetVariableCursorPosition();
            FindAndConvertLineInTreeDOM(ConvertLineTextToList, "find");
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

        /*============ Header of N-th order ============*/
        if(wit_console_command[0]==="h"){
            /*----- <h1>...<h6> -----*/
            if(
                wit_console_command.length === 2 &&
                ["0", "1","2","3","4","5","6"].includes(wit_console_command[1])
            ){
                FindAndConvertLineInTreeDOM(ConvertLineTextToHeader, wit_console_command[1]);
            }
        }
        /*============ List ============*/
        else if(wit_console_command[0]==="l"){
            /*----- dot, number -----*/
            if(
                wit_console_command.length === 2 &&
                ["0", "d", "n"].includes(wit_console_command[1])
            ){
                FindAndConvertLineInTreeDOM(ConvertLineTextToList, wit_console_command[1]);
            }
        }
        else if(wit_console_command[0]==="0"){
            if(wit_console_command.length===1){
                FindAndConvertLineInTreeDOM(ConvertLineTextDeleteStyle);
            }
        }

        SelectFocusElementSetVarTreeAndCurPos("block_write");
    }


    //---------------------------------------------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Find Line in TreeWalker and Function Convert Call  ---
    function FindAndConvertLineInTreeDOM(CallFunc: (node_tree: Node, special_parameter: string)=> void, parameter_func: string = ""): void {
        SetVariableTreeWalker();

        let count_symbol: number = 0;
        let current_node: Node | null;

        while(g_tree_block_write?.nextNode())
        {
            current_node = g_tree_block_write.currentNode ?? null;
            let length_str_current_node = current_node.nodeValue?.length ?? 0;

            if(count_symbol+length_str_current_node >= g_cursor_pos_symbol.position)
            {
                let new_range: Range = document.createRange();
                if(g_cursor_pos_symbol.start)
                {
                    g_tree_block_write?.nextNode();
                    current_node = g_tree_block_write.currentNode ?? null;
                }

                /*-----------------------------------*/
                CallFunc(current_node, parameter_func);
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

    //====================================================================================================
    //================================================================================ Handle Function ===

    //------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Input ---
    function HandleBlurInput(): void {
        // g_mini_console_El.value = "";
    }
    
    //------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Block Write ---
    function HandleInputBlockWrite(): void {
        // CheckTextListDot
    }
    
    
    //======================================================================================================
    //================================================================================ Convert Line Text ===

    //--------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- DEL ALL Style ---
    function ConvertLineTextDeleteStyle(node_tree: Node): void{
        if(node_tree.parentElement){
            let node_content = node_tree?.textContent ?? "";
            let parent_element = node_tree.parentElement;

            parent_element.style.fontSize = "16px";
            parent_element.style.fontWeight = "300";
            parent_element.style.paddingLeft = "0px";
            if(node_content[0]==="•"){
                if(node_content[1]===" "){
                    parent_element.textContent = parent_element.textContent.slice(2);
                }
                else{
                    parent_element.textContent = parent_element.textContent.slice(1);
                }
            }
        }
    }

    //-------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Header ---
    function ConvertLineTextToHeader(node_tree: Node, order_header: string): void {
        if(node_tree.parentElement){
            let parent_element: HTMLElement = node_tree.parentElement;
            if(order_header==="0"){
                parent_element.style.fontSize = "16px";
                parent_element.style.fontWeight = "300";
            }
            else if(order_header==="1"){
                parent_element.style.fontSize = "40px";
                parent_element.style.fontWeight = "600";
            }
            else if(order_header==="2"){
                parent_element.style.fontSize = "32px";
                parent_element.style.fontWeight = "600";
            }
            else if(order_header==="3"){
                parent_element.style.fontSize = "24px";
                parent_element.style.fontWeight = "600";
            }
            else if(order_header==="4"){
                parent_element.style.fontSize = "20px";
                parent_element.style.fontWeight = "600";
            }
            else if(order_header==="5"){
                parent_element.style.fontSize = "16px";
                parent_element.style.fontWeight = "600";
            }
            else if(order_header==="6"){
                parent_element.style.fontSize = "14px";
                parent_element.style.fontWeight = "600";
            }
        }
    }

    //-----------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- List ---
    function ConvertLineTextToList(node_tree: Node, order_header: string): void {
        if(node_tree.parentElement){
            let parent_element = node_tree.parentElement;
            if(order_header==="0")
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
            else if(order_header==="d")
            {
                parent_element.style.paddingLeft = "20px";
                parent_element.textContent = "•" + " " + parent_element.textContent;
            }
            else if(order_header==="n")
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
            else if(order_header==="find")
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


    //---------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Bold & Italic Font
    function ConvertTextToBoldFont(): void {

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
        on:input={HandleInputBlockWrite}
    >
        <div>
            I welcome you to a winter&rsquo;s fairy tale &#10052;
        </div>
    </div>

    <input
        class="mini_console"
        type="text"
        placeholder="CTRL + SHIFT"
        on:blur={HandleBlurInput}
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
