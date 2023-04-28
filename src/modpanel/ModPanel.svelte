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
</div>

<style>
	button.circle {
		border: none;
		margin: 0;
		margin-left: 0.125em;
	}
</style>
