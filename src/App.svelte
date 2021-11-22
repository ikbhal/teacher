<script>
import {onMount} from 'svelte';
//https://svelte.dev/tutorial/writable-stores
import {writable} from 'svelte/store';
// import EnvTest from './EnvTest.svelte';

import {db} from './firebase';
let todayDateString = new Date().toISOString().slice(0,10);
export let date= todayDateString;
let name = "";
let email = "";
let mobile = "";
let skills = "";
let availability = ""; //todo refine to time slot, preferably on satur and sunday, week day early monring 6 to 8 , or 6to 8 evening
export let teachList=writable([]);

// https://vim.fandom.com/wiki/Cut/copy_and_paste_using_visual_selection -> V visual select line, d delete, y copy, p paste
async function loadTeachList(){
	let collRef = db.collection('teacher');
  	let allDocs= await collRef.get();
	let teachListCopy = [];
	for(const doc of allDocs.docs){
		let docData = doc.data();
		let teacherCopy = {
			id:doc.id, 
			date:docData.date, 
			name: docData.name, 
			email: docData.email,
			mobile: docData.mobile,
			skills: docData.skills,
			availability: docData.availability
		};
		teachListCopy.push(teacherCopy);
   		console.log(doc.id, '=>', doc.data());
  	}
	teachList.set(teachListCopy);
}

onMount(async () => {
	loadTeachList();
});

function handleOnSubmit() {
	console.log("form submitted");
	db.collection("teacher").doc(date).set({
	    date,
		name,
		email,
		mobile,
		skills,
		availability
	})
	.then(() => {
		console.log("Document successfully written!");
		loadTeachList();
	})
	.catch((error) => {
		console.error("Error writing document: ", error);
	});
}

function deleteTeacher(id){
	console.log("remove teacher id:", id);
	db.collection("teacher").doc(id).delete().then(() => {
		console.log("Document successfully deleted!");
		loadTeachList();
	}).catch((error) => {
		console.error("Error removing document: ", error);
	});
}
</script>

<main>
	<!-- <EnvTest></EnvTest> -->
<h1>Teacher CRUD</h1>
<p>1. add 2.list 3.delete</p>

<form on:submit|preventDefault={handleOnSubmit}>
	
	<h3>Add Teacher</h3>
	<label>Date:<input type="date" bind:value={date}/></label>
	<label><input type="text" bind:value={name}/>Name</label>
	<label><input type="text" bind:value={email}/>Email</label>
	<label><input type="text" bind:value={mobile}/>Mobile</label>
	<label><input type="text" bind:value={skills}/>Skills</label>
	<label><input type="text" bind:value={availability}/>Availability</label>
	<button type="submit">Save</button>
</form>

<hr/>
<h3>List Teacher</h3>
<div class="teachList">
	{#each $teachList as teacher}
		<div>
			{teacher.date} <br/>
			{teacher.name} <br/>
			{teacher.email} <br/>
			{teacher.mobile} <br/>
			{teacher.skills} <br/>
			{teacher.availability} <br/>
			<button on:click={deleteTeacher(teacher.id)}>Delete</button>
		</div>
	{/each}
</div>
</main>

<style>

</style>