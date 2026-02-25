<script lang="ts">
    import { onMount } from "svelte";
    let g_block_write_El: HTMLDivElement;
    let g_block_console_El: HTMLInputElement;
    let g_tree_block_write: TreeWalker;
    let g_local_rang_block_write: Range;

    /*==================================================*/
    /*=== Interface ====================================*/


    //==============================================================================================
    //================================================================================ Load Page ===
    function LoadElementDOM(): void {
        g_block_write_El = document.querySelector(".block_write") as HTMLDivElement;
        g_block_console_El = document.querySelector(".block_console") as HTMLInputElement;
        g_tree_block_write=SetTreeWalkerForBlockWrite();
        // g_block_console_El.addEventListener("keydown", EnteredCommandIntBlockConsole);
    }
    onMount(LoadElementDOM);

    function SetTreeWalkerForBlockWrite(): TreeWalker{
        return document.createTreeWalker(
            g_block_write_El,
            NodeFilter.SHOW_ELEMENT,
            {
                acceptNode(node) {
                    if(node instanceof HTMLDivElement){
                        if(node.parentNode==g_block_write_El){
                            return NodeFilter.FILTER_ACCEPT
                        }
                        else return NodeFilter.FILTER_REJECT;
                    }
                    else{
                        return NodeFilter.FILTER_REJECT;
                    }
                }
            }
        );
    }
    
    //===========================================================================================================
    //================================================================================ Hotkey & Walk Tree DOM === 

    // When you press the "hotkey", switch between blocks: block_write & block_console
    function CheckHotkey(event: KeyboardEvent){
        // Key down: CTRL + SHIFT;
        // if(event.ctrlKey && event.shiftKey)
        if(event.code === "Tab")
        {
            //just exists
            event.preventDefault();  //overrides the browser's default action
            event.stopPropagation(); //prevents the event from bubbling up to parent elements

            let block_active: HTMLDivElement | HTMLInputElement =document.activeElement as HTMLDivElement | HTMLInputElement;
            
            if(block_active!.className.slice(0, "block_write".length)=="block_write")
            {
                SetLocalRangBlockWrite();
                g_block_console_El.focus();
            }
            else if(block_active!.className.slice(0, "block_console".length)=="block_console" || block_active==null){
                g_block_write_El.focus();
                SetCursorInBlockWrite();
            }
            else{
                g_block_write_El.focus();
                SetCursorInBlockWrite();
            }
        }
    }
    window.addEventListener("keydown", CheckHotkey);

    function SetLocalRangBlockWrite(): void{
        const selection = window.getSelection();
        g_local_rang_block_write=selection!.getRangeAt(0);
    }

    function SetCursorInBlockWrite(): void{
        let range_block_write: Range=document.createRange();
        range_block_write.collapse(true);
        range_block_write.setStart(g_local_rang_block_write.startContainer, g_local_rang_block_write.startOffset);

        const update_sel = window.getSelection();
        if (update_sel) {
            update_sel.removeAllRanges();
            update_sel.addRange(range_block_write);
        }

    }

    //-----------------------------------------------------------------------------------------------------------------------
    //-------------------------------------------------------------------------------- Enter Text Into Block Mini Console ---
    function EnteredCommandIntBlockConsole(event: KeyboardEvent): void {
        if(event.code!=="Enter") return;
        
        event.preventDefault();
        event.stopPropagation();

        let wit_console_command: string = g_block_console_El.value;

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
        <div>Myths are ancient, timeless tales,</div><div><br></div><div>Of gods and heroes, monsters, and whales.</div><div><br></div><div>They tried to explain the world's creation,</div><div><span style="font-weight: 600">Non-fiction</span></div><div><span style="font-weight: 600">And</span> <span style="font-weight: 600"> teach a <span style="font-weight: 800">les-son to </span> every</span> nation.</div><div>More <span style="font-weight: 600"><span style="font-weight: 600">than <span style="font-weight: 800">just</span> stories <span style="font-weight: 600">from</span></span></span> long ago,</div><div>They show us truths that we all know.</div>
        <!-- <div>Myths are ancient, timeless tales,</div><div><br></div><div>Of gods and heroes, monsters, and whales.</div><div><br></div><div>They tried to explain the world's creation,</div><div><span style="font-weight: 600">teach</span></div><div>And teach a les-son to every nation.</div><div>More than just stories from long ago,</div><div>They show us truths that we all know.</div> -->
    </div>

    <input
        class="block_console"
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
    
    .block_console{
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
