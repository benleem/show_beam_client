<script lang="ts">
	import type { Show, ShowFormData } from "$lib/models/shows";
	import { goto } from "$app/navigation"
	import { deleteShow, patchShow, postShow } from "$lib/utils/api/shows";
    import { shows } from "$lib/stores/shows";
	import TextInput from "./Inputs/TextInput.svelte"
	import RadioInput from "./Inputs/RadioInput.svelte"

    export let mode: "add" | "edit" | "delete" 
    export let modal: boolean
    export let show: Show

    let formLoading = false
    let formError = ""

    let showFormData: ShowFormData = {
        title: show.title ,
        description:  show.description ,
        view_code: show.view_code ,
        public: show.public
    }

    const handleSubmit = async() => {
        formLoading = true
        const response = mode === "add" ?  await postShow(showFormData) : await patchShow(showFormData, show.id)
        if (response.show === null || response.status !== "success") {
            formError = response.status
        }
        if (response.show){
            const showResponse = response.show as Show
            if (mode === "add") {
                goto(`/shows/${showResponse.id}`)
            }
            if (mode === "edit") {
                shows.update(shows => {
                    shows.forEach((show: Show, i) => { 
                        if (show.id === showResponse.id) {
                            shows[i] = showResponse;
                        }
                    })
                    return shows
                })
            }
            modal = false
        }
        formLoading = false
    }

    let handleDelete = async() => {
        formLoading = true
        const response = await deleteShow(show.id)
        if (response.show === null || response.status !== "success") {
            formError = response.status
        }
        if (response.show){
            const showId = response.show
            shows.update(shows => {
                const updatedShows = shows.filter((show: Show) => show.id !== showId )
                return updatedShows
            })
            modal = false
        }
        formLoading = false
    }
</script>

<style>

</style>

{#if mode === "delete"}
    <form class="p-5 flex flex-col bg-[#030303] rounded-md border-[1px] border-gray-600" on:submit|preventDefault={() => handleDelete()}>
        <div class="pb-2 flex justify-between border-b-[1px] border-gray-600">
            <h3 class="text-lg">Delete show</h3>
            <button 
                on:click={() => {modal = false}}
                class="material-symbols-outlined bg-gray-600 rounded hover:bg-gray-500 transition-colors duration-200 ease-in-out"
                >
                close
            </button>
        </div>
        <p class="pt-2">Are you sure you want to delete this show?</p>
        <p><span class="text-gray-400">Title:</span> {show.title}</p>
        <p><span class="text-gray-400">Description:</span> {show.description}</p>
        <div class="pt-2">
            <button type="submit" class="w-full p-2 bg-red-500 hover:bg-red-600 rounded">Delete</button>
            {#if formError}
                <p class="pt-1 text-sm text-red-500">{formError}</p>
            {/if}
        </div>
    </form>
{:else}
     <form class="p-5 flex flex-col gap-2 bg-[#030303] rounded-md border-[1px] border-gray-600" novalidate autocomplete="off" on:submit|preventDefault={() => handleSubmit()}>
        <div class="pb-2 flex justify-between border-b-[1px] border-gray-600">
            <h3 class="text-lg">{mode === "add" ? "Add show" : "Edit show details"}</h3>
            <button 
                on:click={() => {modal = false}} 
                class="material-symbols-outlined bg-gray-600 rounded hover:bg-gray-500 transition-colors duration-200 ease-in-out"
                >
                close
            </button>
        </div>
        <TextInput label="Title" name="title" bind:value={showFormData.title}/>
        <TextInput label="Description" name="description" bind:value={showFormData.description}/>
        <TextInput label="View Code" name="viewCode" bind:value={showFormData.view_code}/>
        <fieldset>
            Public?<br>
            <RadioInput label="Yes" name="public" value={true} bind:group={showFormData.public}/>
            <RadioInput label="No" name="public" value={false} bind:group={showFormData.public}/>
        </fieldset>
        <div>
            <button type="submit" class="w-full p-2 bg-green-500 hover:bg-green-600 rounded">Submit</button>
            {#if formError}
                <p class="pt-1 text-sm text-red-500">{formError}</p>
            {/if}
        </div>
    </form>
{/if}
{#if formLoading}
    <div class="absolute top-0 left-0 h-full w-full grid place-content-center bg-white opacity-40 animate-pulse">
        <p>Loading</p>
    </div>
{/if}