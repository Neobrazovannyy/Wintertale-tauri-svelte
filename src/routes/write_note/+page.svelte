<script lang="ts">
    let if_convert_write: boolean = $state(true);
    let text_textarea: string = $state("");

    function ShowBlockRide(): void{
        if_convert_write=false;
    }

    function ShowBlockWrite(): void{
        if_convert_write=true;
    }

    //! doesn't wark
    function GetTextFromTextarea(): void{
        let el_textarea = document.querySelector(".block_write") as HTMLTextAreaElement;
        let text_textarea = el_textarea.value;
    }

</script>

<div class="record_field">
    {#if if_convert_write}
        <textarea class="block_write"
            on:blur={ShowBlockRide}
            on:input={GetTextFromTextarea}
            autofocus
        ></textarea>
    {:else}
        <div class="block_rite"
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
    $bg_dark_grey: #161b24; // first choice
    $bg_dark_blue: #172030; // second choice

    *{
        border: 2px solid #000; //del
    }

    .record_field{
        display: flex;
        flex-direction: column;
        align-items: center;
        width: 100%;
        height: 100%;
    }

    @mixin style_block_write_rite{
        font-size: 16px;
        letter-spacing: 1.6px;
        font-family: $absolute_font_family;
        font-weight: $absolute_font_weight;
        width: 794px;
        height: 100%;
        padding: 5px;
    }

    .block_write{
        @include style_block_write_rite;
        resize: none;
    }

    .block_rite{
        @include style_block_write_rite;
    }

</style>
