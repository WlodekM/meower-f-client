<script>
	import Container from "../lib/Container.svelte";
	import {
		mainPage as page,
		user,
		profileClicked,
		chatid,
		modalShown,
		modalPage,
		lastTyped,
	} from "../lib/stores.js";
	
	import {link} from "../lib/clmanager.js"
	import {tick} from "svelte";
	import {fade} from "svelte/transition";
	import {shiftHeld} from "../lib/keyDetect.js";
	
	import {profileCache} from "../lib/loadProfile.js";
	import {autoresize} from "svelte-textarea-autoresize";

	import ProfileView from "../lib/ProfileView.svelte";

	import PFP from "../lib/PFP.svelte";
	import Loading from "../lib/Loading.svelte";
	import * as clm from "../lib/clmanager.js";
	import {apiUrl, encodeApiURLParams} from "../lib/urls.js";

	const PFP_COUNT = 38;
	var pfp_temp = $user.pfp_data
	var result_eval, code_eval

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
	let postErrors = "";
	
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
	var whusername = "User"
	var whpost = ""
	function whsend() {
		console.log(`${whusername}:${whpost}`)
		fetch('https://webhooks.meower.org/post/home', {
			method: 'POST',
			headers: {
				'Accept': 'application/json',
				'Content-Type': 'application/json'
			},
			body: JSON.stringify({ "post": whpost,"username":whusername })
		})
		.then(response => response.text())
		.then(response => console.log(response))
	}

</script>

<div class="ModPanel">
	<Container>
		<h1>Developer tools</h1>
		Hello everyone its THE DEV TOOLS HERE back with another youtube video!!!!!!
	</Container>
	<Container style="height: 150px;">
		<div class="settings-controls">
            <input bind:value={whusername} type="text" id="username" style="margin-bottom:4px">
        	<textarea bind:value={whpost} rows="4" class="container type-message" style="resize: none;width:calc(100% - (11px * 2) - 100px)"></textarea>
			<button
				class="circle settings"
				on:click={() => {
					whsend()
				}}
			/>
		</div>
		<h1>Webhook</h1>
		Send message using webhooks.<br><br><br><br>
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
<!-- 	<Container>
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
	</Container> -->
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
	<Container>
		<h1>Evaluate</h1>
		Run code (Dangerous)
		<form
				class="createpost"
				autocomplete="off"
				on:submit|preventDefault={e => {
					postErrors = "";
					if (!e.target[0].value.trim()) {
						postErrors = "You can't eval an empty string!";
						return false;
					}
					try {
						result_eval = eval(e.target[0].value)
						console.log(`Code: ${e.target[0].value}`)
					} catch(err) {
						result_eval = err
					}
				}}
			>
				<textarea
					type="text"
					class="white"
					placeholder="Write something..."
					id="postinput"
					name="postinput"
					autocomplete="false"
					rows="1"
					use:autoresize
					on:input={() => {
						if ($lastTyped + 1500 < +new Date()) {
							lastTyped.set(+new Date());
							link.send({
								cmd: "direct",
								val: {
									cmd: "set_chat_state",
									val: {
										chatid: "livechat",
										state: 101,
									},
								},
								listener: "typing_indicator",
							});
						}
					}}
					on:keydown={event => {
						if (event.key == "Enter" && !shiftHeld) {
							event.preventDefault();
							document.getElementById("submitpost").click();
						}
					}}
					bind:this={code_eval}
				/>
		<div class="post-errors">{postErrors}</div>
		<div class="settings-controls">
			<button
				class="circle settings"
				alt="Go!" id="submitpost"
			/>
		</div>
		
		<input disabled bind:value={result_eval}>
			</form>
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
