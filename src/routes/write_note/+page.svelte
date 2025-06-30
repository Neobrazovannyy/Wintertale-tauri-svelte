<script lang="ts">
    let m_if_convert_write: boolean = $state(true);
    let m_text_textarea: string = $state("");

    function ShowBlockRead(): void{
        m_if_convert_write=false;
    }

    function ShowBlockWrite(): void{
        m_if_convert_write=true;
    }

    async function HandleBlurBlockWrite(): Promise<void>{
        await ShowBlockRead();
        await SetTextBlockRead();
    }

    function GetTextFromBlockWrite(): void{
        let el_block_write = document.querySelector(".block_write") as HTMLTextAreaElement | null;
        let text_textarea: string | undefined = el_block_write?.value;
        if(typeof(text_textarea) == "undefined"){
            m_text_textarea="";
        }
        else if(typeof(text_textarea) == "string"){
            m_text_textarea=text_textarea;
        }
    }

    function SetTextBlockRead(): void{
        let el_block_read = document.querySelector(".block_read") as HTMLDivElement | null;
        if(el_block_read){
            // el_block_read.innerHTML=m_text_textarea;
            el_block_read.innerText=m_text_textarea;
        }
    }

    function SetTextBlockWrite(): void{
        let el_block_write = document.querySelector(".block_write") as HTMLTextAreaElement | null;
        if(el_block_write){
            el_block_write.value = m_text_textarea;
        }
    }

</script>

<div class="record_field">
    {#if m_if_convert_write}
        <textarea class="block_write"
            on:focus={SetTextBlockWrite}
            on:blur={HandleBlurBlockWrite}
            on:input={GetTextFromBlockWrite}
            autofocus
        ></textarea>
    {:else}
        <div class="block_read"
            on:click={ShowBlockWrite}
        ></div>
    {/if}
</div>
<!-- 
A4:
    width: 794px,
    height: 1123px
-->

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

    @mixin style_block_write_read{
        font-size: 16px;
        letter-spacing: 1.6px;
        font-family: $absolute_font_family;
        font-weight: $absolute_font_weight;
        background-color: $bg_more_dark_grey;
        width: 794px;
        height: 100%;
        padding: 5px;
        margin: 5px;
        border: 0px;
        // border: 0px solid $text_and_border_lite_blue;
        border-radius: 5px;
        scrollbar-width: thin;
        scrollbar-gutter: stable;
    }

    /*===================*/
    /*=== Block Write ===*/
    /*===================*/

    .block_write{
        @include style_block_write_read;
        resize: none;
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

    /*===================*/
    /*=== Block Read ===*/
    /*===================*/

    .block_read{
        @include style_block_write_read;
        word-wrap: break-word;
        overflow-y: scroll;
    }


    .block_read::-webkit-scrollbar {
        width: 5px;
        height: 10px;
        
        &:hover{
            width: 10px;
        }
    }

    .block_read::-webkit-scrollbar-track {
        background: #00000000;
    }

    .block_read::-webkit-scrollbar-thumb {
        background-color: rgba(229, 240, 255, 0.2);
        border-radius: 3px;
        border: 0px;

        &:hover{
            background-color: rgba(229, 240, 255, 0.6);
        }
    }

</style>
