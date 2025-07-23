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
    }
    onMount(LoadElementDOM);

    //================================================================================ Hotkey & Rendering
    
    // If you click on a field (block_write), it is in the focus variable (g_focus_block)
    function ClickFieldBlockWrite(): void {
        if(g_focus_block==="mini_console")
        {
            g_focus_block = "block_write";   //? document.activeElement
        }
    }
    
    // Key down: CTRL + SHIFT + SPACE;
    function CheckHotkey(event: KeyboardEvent) {
        if(event.ctrlKey && event.shiftKey && event.code==="Space")
        {
            // focus moves to the object: "mini console"
            if(g_focus_block==="block_write")
            {
                SelectFocusElement(false);
            }
            // focus moves to the object: "block write"
            else if(g_focus_block==="mini_console")
            {
                SelectFocusElement(true);
            }

        }
    }
    window.addEventListener("keydown", CheckHotkey);


    //-------------------------------------------------------------------------------- Focus Element
    function SelectFocusElement(focus_element: boolean): void {
        // true => "block write"
        if(focus_element){
            g_block_write_El.focus();
            SetCursorPosition();
            g_focus_block="block_write";
        }
        // false => "mini console"
        else{
            SetVariableTreeWalker();
            SetVariableCursorPosition();
            // SetVariableTextSelect();
            g_mini_console_El.value="";
            g_mini_console_El.focus();
            g_focus_block = "mini_console";
        }
    }


    //-------------------------------------------------------------------------------- Enter Text Into Mini Console
    function EnteredCommandIntoMiniConsole(event: KeyboardEvent): void {
        if(event.code!=="Enter") return;

        SetVariableTreeWalker();
        
        event.preventDefault();
        event.stopPropagation();

        let arr_console_command: string[] = g_mini_console_El.value.split(" ");

        /*============ Header of N-th order ============*/
        if(arr_console_command[0]==="h"){
            /*----- <h1>...<h6> -----*/
            if(
                arr_console_command.length != 2 ||
                !["1","2","3","4","5","6"].includes(arr_console_command[1])
            ){
                SelectFocusElement(true);
                g_mini_console_El.value = "help: h [1..6]";
            }
            else{
                ConvertTextToHeader(arr_console_command[1]);
            }
        }
        /*============ Command error ============*/
        else{
            SelectFocusElement(true);
            g_mini_console_El.value = "Command was not found";
        }

        SelectFocusElement(true);
    }


    //-------------------------------------------------------------------------------- Set Global Variable

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

    // Set the cursor
    function SetCursorPosition(cursor_position: PositionStartLine = g_cursor_pos_symbol, element_focus: Element = g_block_write_El): void {
        let count_symbol: number = 0;
        let current_node: Node | null;

        while(g_tree_block_write?.nextNode()){
            current_node = g_tree_block_write.currentNode ?? null;
            let length_str_current_node = current_node.nodeValue?.length ?? 0;

            if(count_symbol+length_str_current_node >= cursor_position.position)
            {
                let new_range: Range = document.createRange();
                new_range.collapse(true);
                if(cursor_position.start)
                {
                    g_tree_block_write?.nextNode();
                    current_node = g_tree_block_write.currentNode ?? null;         
                    
                    new_range.setStart(current_node, 0);
                }
                else{
                    new_range.setStart(current_node, cursor_position.position-count_symbol);
                }

                let select_el = window.getSelection();
                if(select_el){
                    select_el.removeAllRanges();
                    select_el.addRange(new_range);
                }

                return;
            }
            count_symbol+=length_str_current_node;
        }
    }

    // Create TreeWalker
    function SetVariableTreeWalker(element_focus: Element = g_block_write_El): void {
        g_tree_block_write = document.createTreeWalker(
            element_focus,
            NodeFilter.SHOW_TEXT,
            null
        );
    }

    
    //================================================================================ Handle Function

    //-------------------------------------------------------------------------------- Input
    function HandleBlurInput(): void {
        g_mini_console_El.value = "";
    }
    
    //-------------------------------------------------------------------------------- Block Write
    function HandleInputBlockWrite(): void {
        // let text_content: string = g_block_write_El.innerText?.slice(-4) ?? "";

        // if(text_content == "\n - "){
        //     L("ConvertListDot");
        //     ConvertListDot();
        // }
    }
    
    
    //================================================================================ Convert Text

    //-------------------------------------------------------------------------------- Header
    function ConvertTextToHeader(order_header: string): void {
        // let count_symbol: number = 0;
        // let current_node: Node | null;

        // while(g_tree_block_write?.nextNode()){
        //     current_node = g_tree_block_write.currentNode ?? null;
        //     let length_str_current_node = current_node.nodeValue?.length ?? 0;

        //     if(count_symbol+length_str_current_node >= g_cursor_pos_symbol.position)
        //     {
        //         let new_range: Range = document.createRange();
        //         new_range.collapse(true);
        //         if(g_cursor_pos_symbol.start)
        //         {
        //             g_tree_block_write?.nextNode();
        //             current_node = g_tree_block_write.currentNode ?? null;         
                    
        //             new_range.setStart(current_node, 0);
        //         }
        //         else{
        //             new_range.setStart(current_node, g_cursor_pos_symbol.position-count_symbol);
        //         }

        //         let select_el = window.getSelection();
        //         if(select_el){
        //             select_el.removeAllRanges();
        //             select_el.addRange(new_range);
        //         }

        //         return;
        //     }
        //     count_symbol+=length_str_current_node;
        // }

    }

    //-------------------------------------------------------------------------------- Bold & Italic Font
    function ConvertTextToBoldFont(): void {

    }

    //-------------------------------------------------------------------------------- List Dot
    function ConvertListDot(cursor_position: PositionStartLine = g_cursor_pos_symbol): void {   // '\n' + ' ' + '-' + ' ' => char[4]
    }

    
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
    ></div>

    <input
        class="mini_console"
        type="text"
        placeholder="CTRL+SHIFT+SPACE"
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

        &:focus{
            background-color: $text_and_border_lite_blue;
            outline: none;
            box-shadow: none;
        }
    }

</style>
