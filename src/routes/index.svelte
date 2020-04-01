<script>
  import { onMount } from "svelte";
  import { scale, fade } from "svelte/transition";
  import { flip } from "svelte/animate";
  import meetups from "../meetups-store.js";
  import MeetupItem from "../components/Meetup/MeetupItem.svelte";
  import MeetupFilter from "../components/Meetup/MeetupFilter.svelte";
  import Button from "../components/UI/Button.svelte";
  import EditMeetup from "../components/Meetup/EditMeetup.svelte";
  import LoadingSpinner from "../components/UI/LoadingSpinner.svelte";

  let _editMode;
  let _editedId;
  let _isLoading;
  let meetupsPromise;

  onMount(() => {
    meetupsPromise = fetch(
      "https://svelte-meetups-635c8.firebaseio.com/meetups.json"
    )
      .then(res => {
        if (!res.ok) {
          throw new Error("Fetching meetups failed, please try again later!");
        }
        return res.json();
      })
      .then(data => {
        const loadedMeetups = [];
        for (const key in data) {
          loadedMeetups.push({
            ...data[key],
            id: key
          });
        }
        meetups.setMeetups(loadedMeetups.reverse());
        // equivalent to reset the values fetching from the server
      });
  });

  let _favsOnly = false;

  $: filteredMeetups = _favsOnly
    ? $meetups.filter(m => m.isFavorite)
    : $meetups;

  function setFilter(event) {
    _favsOnly = event.detail === 1;
  }

  function savedMeetup(event) {
    _editMode = null;
    _editedId = null;
  }

  function cancelEdit() {
    _editMode = null;
    _editedId = null;
  }

  // function showDetails(event) {
  //   _page = "details";
  //   _id = event.detail;
  // }

  // function closeDetails() {
  //   _page = "overview";
  //   _id = undefined;
  // }

  function startEdit(event) {
    _editMode = "edit";
    _editedId = event.detail;
  }

  function startAdd() {
    _editMode = "edit";
  }

  function clearError() {
    window.location.reload(true);
  }
</script>

<style>
  #meetups {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr;
    grid-gap: 1rem;
  }

  #meetup-controls {
    margin: 1rem;
    display: flex;
    justify-content: space-between;
  }

  #no-meetups {
    margin: 1rem;
  }

  @media (min-width: 768px) {
    #meetups {
      grid-template-columns: repeat(2, 1fr);
    }
  }
</style>

<svelte:head>
  <title>Sapper-meetups</title>
</svelte:head>

{#if _editMode === 'edit'}
  <EditMeetup on:save={savedMeetup} on:cancel={cancelEdit} id={_editedId} />
{/if}
{#await meetupsPromise}
  <LoadingSpinner />
{:then value}
  <section id="meetup-controls">
    <MeetupFilter on:select={setFilter} />
    <Button on:click={startAdd}>New meetup</Button>
  </section>
  <section id="meetups">
    {#if filteredMeetups.length === 0}
      <p id="no-meetups" in:fade={{ delay: 400 }}>No meetups found</p>
    {:else}
      {#each filteredMeetups as meetup (meetup.id)}
        <div animate:flip={{ duration: 200 }} transition:scale>
          <MeetupItem
            id={meetup.id}
            title={meetup.title}
            subtitle={meetup.subtitle}
            imageUrl={meetup.imageUrl}
            description={meetup.description}
            email={meetup.contactEmail}
            address={meetup.address}
            isFav={meetup.isFavorite}
            on:edit={startEdit} />
        </div>
      {/each}
    {/if}
  </section>

{:catch error}
  <Error message={error.message} on:cancel={clearError} />
{/await}
