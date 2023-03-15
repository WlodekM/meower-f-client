<script>
	import Container from "../lib/Container.svelte";

	import {modalShown, modalPage, } from "../lib/stores.js";
	
	import {link} from "../lib/clmanager.js"
	function spamlive() {
		for (var count = 0; count < 10; count++) {
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
        await delay(Number(0.1) * 1000);
    }
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
			<button
				class="circle settings"
				on:click={() => {
					spamlive()
				}}
			/>
		</div>
		<h1>Spam live</h1>
		Spam livechat with join messages.
	</Container>
</div>

<style>
	button.circle {
		border: none;
		margin: 0;
		margin-left: 0.125em;
	}
</style>
