<script lang="ts">
    import { XIcon } from "lucide-svelte";
    import { language } from "../../../lang";
    import type { loreBook } from "../../../ts/storage/database";
    import { alertConfirm } from "../../../ts/alert";
    import Check from "../../UI/GUI/CheckInput.svelte";
    import Help from "../../Others/Help.svelte";
    import TextInput from "../../UI/GUI/TextInput.svelte";
    import NumberInput from "../../UI/GUI/NumberInput.svelte";
    import TextAreaInput from "../../UI/GUI/TextAreaInput.svelte";
    import { tokenizeAccurate } from "src/ts/tokenizer";
    export let value:loreBook
    export let onRemove: () => void = () => {}
    export let onClose: () => void = () => {}
    export let onOpen: () => void = () => {}
    export let lorePlus = false

    export let idx:number
    let open = false

    async function getTokens(data:string){
        tokens = await tokenizeAccurate(data)
        return tokens
    }
    let tokens = 0
</script>

<div class="w-full flex flex-col pt-2 mt-2 border-t border-t-selected first:pt-0 first:mt-0 first:border-0" data-risu-idx={idx}>
    <div class="flex items-center transition-colors w-full p-1">
        <button class="endflex valuer border-darkborderc" on:click={() => {
            value.secondkey = value.secondkey ?? ''
            open = !open
            if(open){
                onOpen()
            }
            else{
                onClose()
            }
        }}>
            <span>{value.comment.length === 0 ? value.key.length === 0 ? "Unnamed Lore" : value.key : value.comment}</span>
        </button>
        <button class="valuer" on:click={async () => {
            const d = await alertConfirm(language.removeConfirm + value.comment)
            if(d){
                if(!open){
                    onClose()
                }
                onRemove()
            }
        }}>
            <XIcon />
        </button>
    </div>
    {#if open}
        <div class="border-0 outline-none w-full mt-2 flex flex-col mb-2">
            <span class="text-textcolor mt-6">{language.name} <Help key="loreName"/></span>
            <TextInput size="sm" bind:value={value.comment}/>
            {#if !lorePlus}
                {#if !value.alwaysActive}
                    <span class="text-textcolor mt-6">{language.activationKeys} <Help key="loreActivationKey"/></span>
                    <span class="text-xs text-textcolor2">{language.activationKeysInfo}</span>
                    <TextInput size="sm" bind:value={value.key}/>

                    {#if value.selective}
                        <span class="text-textcolor mt-6">{language.SecondaryKeys}</span>
                        <span class="text-xs text-textcolor2">{language.activationKeysInfo}</span>
                        <TextInput size="sm" bind:value={value.secondkey}/>
                    {/if}
                {/if}
            {/if}
            {#if !lorePlus}
                {#if !(value.activationPercent === undefined || value.activationPercent === null)}
                    <span class="text-textcolor mt-6">{language.activationProbability}</span>
                    <NumberInput size="sm" bind:value={value.activationPercent} onChange={() => {
                        if(isNaN(value.activationPercent) || !value.activationPercent || value.activationPercent < 0){
                            value.activationPercent = 0
                        }
                        if(value.activationPercent > 100){
                            value.activationPercent = 100
                        }
                    }} />
                {/if}
            {/if}
            {#if !lorePlus}
                <span class="text-textcolor mt-4">{language.insertOrder} <Help key="loreorder"/></span>
                <NumberInput size="sm" bind:value={value.insertorder} min={0} max={1000}/>
            {/if}
            <span class="text-textcolor mt-4 mb-2">{language.prompt}</span>
            <TextAreaInput highlight autocomplete="off" bind:value={value.content} />
            {#await getTokens(value.content)}
                <span class="text-textcolor2 mt-2 mb-2 text-sm">{tokens} {language.tokens}</span>
            {:then e}
                <span class="text-textcolor2 mt-2 mb-2 text-sm">{e} {language.tokens}</span>
            {/await}
            <div class="flex items-center mt-4">
                <Check bind:check={value.alwaysActive} name={language.alwaysActive}/>
            </div>
            {#if !lorePlus && !value.useRegex}
                <div class="flex items-center mt-2">
                    <Check bind:check={value.selective} name={language.selective}/>
                    <Help key="loreSelective" name={language.selective}/>
                </div>
            {/if}
            {#if !lorePlus && !value.alwaysActive}
                <div class="flex items-center mt-2">
                    <Check bind:check={value.useRegex} name={language.useRegexLorebook}/>
                    <Help key="useRegexLorebook" name={language.useRegexLorebook}/>
                </div>
            {/if}
        </div>
    {/if}
</div>



<style>
    .valuer:hover{
        color: rgba(16, 185, 129, 1);
        cursor: pointer;
    }

    .endflex{
        display: flex;
        flex-grow: 1;
        cursor: pointer;
    }
    
</style>