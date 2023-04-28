<script>
	import Container from "../lib/Container.svelte";
	import {
		mainPage as page,
		user,
		profileClicked,
		chatid,
		modalShown,
		modalPage,
	} from "../lib/stores.js";
	
	import {link} from "../lib/clmanager.js"
	import {tick} from "svelte";
	import {fade} from "svelte/transition";
	
	import {profileCache} from "../lib/loadProfile.js";

	import ProfileView from "../lib/ProfileView.svelte";

	import PFP from "../lib/PFP.svelte";
	import Loading from "../lib/Loading.svelte";
	import * as clm from "../lib/clmanager.js";
	import {apiUrl, encodeApiURLParams} from "../lib/urls.js";

	const PFP_COUNT = 38;
	var pfp_temp = $user.pfp_data

	const pfps = new Array(PFP_COUNT).fill().map((_, i) => i + 1);
	pfps.push(-1, 500, 101, 102, 404) //add secret pfps
	let pfpSwitcher = false;

	async function loadProfile() {
		let path = `users/${$profileClicked}`;
		if (encodeApiURLParams) path = encodeURIComponent(path);
		const resp = await fetch(`${apiUrl}${path}`);
		if (!resp.ok) {
			throw new Error("Response code is not OK; code is " + resp.status);
		}
		const json = await resp.json();
		return json;
	}

	/**
	 * Saves the user profile, and also clears its cache entry.
	 */
	function save() {
		if ($profileCache[$user.name]) {
			const _profileCache = $profileCache;
			delete _profileCache[$user.name];
			profileCache.set(_profileCache);
		}

		clm.updateProfile();
	}

	let pfpOverflow = false;
	var target_page = "groupcat"
	$: {
		const pfp = $user.pfp_data;
		let hiddenpfps = [-1, 500, 101, 102]
		pfpOverflow = pfp < 0 || pfp > PFP_COUNT && !(hiddenpfps.includes(pfp));
	}
	var spamn = 10
	function spamlive() {
		for (var count = 0; count < spamn; count++) {
        link.send({
            "cmd": "direct",
            "val": {
                "cmd": "set_chat_state",
                "val": {
                    "state": 1,
                    "chatid": "livechat"
                }
            },
        })
    }
	}
	
	function goto(newPage, resetScroll = true) {
		if (
			!$user.name &&
			newPage !== "home" &&
			newPage !== "settings" &&
			newPage !== "changelog" &&
			newPage !== "search"
		) {
			modalPage.set("signup");
			modalShown.set(true);
			return;
		}
		if (resetScroll) {
			window.scrollTo(0, 0);
		}
		if ($page === "groupchat") {
			clm.meowerRequest({
				cmd: "direct",
				val: {
					cmd: "set_chat_state",
					val: {
						state: 0,
						chatid: $chatid,
					},
				},
			});
		}
		chatid.set("");
		page.set("blank");
		tick().then(() => page.set(newPage));
	}
</script>

<div class="ModPanel">
	<Container>
		<h1>Moderation Panel</h1>
		Hello everyone its THE MOD PANEL HERE back with another youtube video!!!!!!
	</Container>
	<Container>
		<div class="settings-controls">
			<button
				class="circle settings"
				on:click={() => {
					modalPage.set("modUserInp");
					modalShown.set(true);
				}}
			/>
		</div>
		<h1>Moderate User</h1>
		Moderate a certain User.
	</Container>
	<Container>
		<div class="settings-controls">
			<p>Start:</p>
			<button
				class="circle settings"
				alt="Start spamming"
				on:click={() => {
					spamlive()
				}}
			/>
			&emsp;
				<p>Configure:</p>
			<button
				class="circle settings"
				alt="Set how many times to spam"
				on:click={() => {
					spamn = window.prompt(`How many times to spam\nCurrent:${spamn}`);
				}}
			/>
		</div>
		<h1>Spam live</h1>
		Spam livechat with join messages.<br><br><br><br><br><br>
	</Container>
	<Container>
		<div class="settings-controls">
			<button
				class="circle settings"
				alt="Go to group cat"
				on:click={() => goto("groupcat")}
			/>
		</div>
		<h1>group cat</h1>
		Groupcat
	</Container>
	{#await loadProfile()}
			<Loading />
	{:then data}
	<Container>
		<div class="settings-controls">
			<button
				class="circle settings"
				alt="Set pfp"
				on:click={() => {
								pfpSwitcher = false;
								$user.pfp_data = pfp_temp;
								save()
								}
						  }
			/>
		</div>
		<h1>Set pfp</h1>
		<input bind:value={pfp_temp}>
	</Container>
	<Container>
		<div class="settings-controls">
			<button
				class="circle settings"
				alt="Go!"
				on:click={() => {
								goto(target_page)
								}
						  }
			/>
		</div>
		<h1>Go to page</h1>
		<input bind:value={target_page}>
	</Container>
	{:catch e}
		<ProfileView username={$profileClicked} />
	{/await}
</div>

<style>
	button.circle {
		border: none;
		margin: 0;
		margin-left: 0.125em;
	}
</style>
