<script lang="ts">
    import { onMount } from "svelte";

onMount(()=>{
    let m_block_write_El: HTMLDivElement = document.querySelector(".block_write") as HTMLDivElement;
    let m_mini_console_El: HTMLInputElement =  document.querySelector(".mini_console") as HTMLInputElement;
    let m_focus_block: HTMLDivElement | HTMLInputElement = $state(m_block_write_El);

    
    //================================================================================ Hotkey
    $effect(()=>{
        window.addEventListener("keydown", (event)=>
        {
            if(event.ctrlKey && event.shiftKey && event.code === "Space")
            {
                // L("==================")
                // L("startContainer", window.getSelection()?.getRangeAt(0)?.startContainer);
                // L("startOffset", window.getSelection()?.getRangeAt(0)?.startOffset);
                // L("endOffset", window.getSelection()?.getRangeAt(0)?.endOffset);
                // L("------------------")
                // L("anchorNode", window.getSelection()?.anchorNode);
                // L("toString", window.getSelection()?.toString());
                // L("endOffset", window.getSelection()?.anchorOffset);
                // L("focusOffset", window.getSelection()?.focusOffset);
                
                L("startContainer", window.getSelection()?.toString());
                L("startContainer", window.getSelection()?.getRangeAt(0)?.startOffset);
                L("startContainer", window.getSelection()?.toString()?.length);

                // focus moves to the object: "mini console"
                if(m_focus_block instanceof HTMLDivElement)
                {
                    m_mini_console_El.focus();
                    m_focus_block = m_mini_console_El;
                }
                // focus moves to the object: "block write"
                else if(m_focus_block instanceof HTMLInputElement)
                {
                    m_block_write_El.focus();
                    // m_block_write_El.selectionStart = m_cursor_pos_textarea;
                    m_mini_console_El.value = "";
                    m_focus_block = m_block_write_El;
                }

            }
        });
    });

});
    //================================================================================ Helper functions
    
    function L(description: string="", variable: unknown=""): void {
        console.log(`${description}: ${variable}`);
    }

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
    ></div>

    <input
        class="mini_console"
        type="text"
        placeholder="CTRL+SHIFT+SPACE"
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
