<script lang="ts">
    import { alertCardExport, alertConfirm, alertError } from "../../ts/alert";
    import { language } from "../../lang";
    import { changeToPreset, copyPreset, downloadPreset, importPreset } from "../../ts/storage/database.svelte";
    import { DBState } from 'src/ts/stores.svelte';
    import { CopyIcon, Share2Icon, PencilIcon, FolderUpIcon, PlusIcon, TrashIcon, XIcon } from "lucide-svelte";
    import TextInput from "../UI/GUI/TextInput.svelte";
    import { prebuiltPresets } from "src/ts/process/templates/templates";
    import { ShowRealmFrameStore } from "src/ts/stores.svelte";

    let editMode = $state(false)
    interface Props {
        close?: any;
    }

    let { close = () => {} }: Props = $props();

</script>

<div class="absolute w-full h-full z-40 bg-black bg-opacity-50 flex justify-center items-center">
    <div class="bg-darkbg p-4 break-any rounded-md flex flex-col max-w-3xl w-96 max-h-full overflow-y-auto">
        <div class="flex items-center text-textcolor mb-4">
            <h2 class="mt-0 mb-0">{language.presets}</h2>
            <div class="flex-grow flex justify-end">
                <button class="text-textcolor2 hover:text-green-500 mr-2 cursor-pointer items-center" onclick={close}>
                    <XIcon size={24}/>
                </button>
            </div>
        </div>
        {#each DBState.db.botPresets as presets, i}
            <button onclick={() => {
                if(!editMode){
                    changeToPreset(i)
                    close()
                }
            }} class="flex items-center text-textcolor border-t-1 border-solid border-0 border-darkborderc p-2 cursor-pointer" class:bg-selected={i === DBState.db.botPresetsId}>
                {#if editMode}
                    <TextInput bind:value={DBState.db.botPresets[i].name} placeholder="string" padding={false}/>
                {:else}
                    {#if i < 9}
                    <span class="w-2 text-center mr-2 text-textcolor2">{i + 1}</span>
                    {/if}
                    <span>{presets.name}</span>
                {/if}
                <div class="flex-grow flex justify-end">
                    <div class="text-textcolor2 hover:text-green-500 cursor-pointer mr-2" role="button" tabindex="0" onclick={(e) => {
                        e.stopPropagation()
                        copyPreset(i)
                    }} onkeydown={(e) => {
                        if(e.key === 'Enter'){
                            e.currentTarget.click()
                        }
                    }}>
                        <CopyIcon size={18}/>
                    </div>
                    <div class="text-textcolor2 hover:text-green-500 cursor-pointer mr-2" role="button" tabindex="0" onclick={async (e) => {
                        e.stopPropagation()
                        const data = await alertCardExport('preset')
                        console.log(data.type)
                        if(data.type === ''){
                            downloadPreset(i, 'risupreset')
                        }
                        if(data.type === 'realm'){
                            $ShowRealmFrameStore = `preset:${i}`
                        }
                    }} onkeydown={(e) => {
                        if(e.key === 'Enter'){
                            e.currentTarget.click()
                        }
                    }}>

                        <Share2Icon size={18} />
                    </div>
                    <div class="text-textcolor2 hover:text-green-500 cursor-pointer" role="button" tabindex="0" onclick={async (e) => {
                        e.stopPropagation()
                        if(DBState.db.botPresets.length === 1){
                            alertError(language.errors.onlyOneChat)
                            return
                        }
                        const d = await alertConfirm(`${language.removeConfirm}${presets.name}`)
                        if(d){
                            changeToPreset(0)
                            let botPresets = DBState.db.botPresets
                            botPresets.splice(i, 1)
                            DBState.db.botPresets = botPresets
                            changeToPreset(0, false)
                        }
                    }} onkeydown={(e) => {
                        if(e.key === 'Enter'){
                            e.currentTarget.click()
                        }
                    }}>
                        <TrashIcon size={18}/>
                    </div>
                </div>
            </button>
        {/each}
        <div class="flex mt-2 items-center">
            <button class="text-textcolor2 hover:text-green-500 cursor-pointer mr-1" onclick={() => {
                let botPresets = DBState.db.botPresets
                let newPreset = safeStructuredClone(prebuiltPresets.OAI2)
                newPreset.name = `New Preset`
                botPresets.push(newPreset)

                DBState.db.botPresets = botPresets
            }}>
                <PlusIcon/>
            </button>
            <button class="text-textcolor2 hover:text-green-500 mr-2 cursor-pointer" onclick={() => {
                importPreset()
            }}>
                <FolderUpIcon size={18}/>
            </button>
            <button class="text-textcolor2 hover:text-green-500 cursor-pointer" onclick={() => {
                editMode = !editMode
            }}>
                <PencilIcon size={18}/>
            </button>
        </div>
        <span class="text-textcolor2 text-sm">{language.quickPreset}</span>
    </div>
</div>

<style>
    .break-any{
        word-break: normal;
        overflow-wrap: anywhere;
    }
</style>