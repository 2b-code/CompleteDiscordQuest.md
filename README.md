## Complete Recent Discord Quest
> [!NOTE]
> This no longer works in browser!

> [!NOTE]
> This no longer works if you're alone in vc! Somebody else has to join you!
>

How to use this script:
1. Accept the quest under User Settings -> Gift Inventory
2. Join a vc
3. **Join the same vc on an alt**
4. Stream any window (can be notepad or something)
5. Press <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>I</kbd> to open DevTools
6. Go to the `Console` tab
7. Paste the following code and hit enter:
```js
let wpRequire;
window.webpackChunkdiscord_app.push([[ Math.random() ], {}, (req) => { wpRequire = req; }]);

let ApplicationStreamingStore = Object.values(wpRequire.c).find(x => x?.exports?.default?.getStreamerActiveStreamMetadata).exports.default;
let QuestsStore = Object.values(wpRequire.c).find(x => x?.exports?.default?.getQuest).exports.default;
let FluxDispatcher = Object.values(wpRequire.c).find(x => x?.exports?.default?.flushWaitQueue).exports.default;

let quest = [...QuestsStore.quests.values()].find(x => x.userStatus?.enrolledAt && !x.userStatus?.completedAt && new Date(x.config.expiresAt).getTime() > Date.now())
let isApp = navigator.userAgent.includes("Electron/")
if(!isApp) {
	console.log("This no longer works in browser. Use the desktop app!")
} else if(!quest) {
	console.log("You don't have any uncompleted quests!")
} else {
	let pid = Math.floor(Math.random() * 30000) + 1000
	ApplicationStreamingStore.getStreamerActiveStreamMetadata = () => ({
		id: quest.config.applicationId,
		pid,
		sourceName: null
	})
	
	let secondsNeeded = quest.config.streamDurationRequirementMinutes * 60
	let fn = data => {
		let progress = data.userStatus.streamProgressSeconds
		console.log(`Quest progress: ${progress}/${secondsNeeded}`)
		
		if(progress >= secondsNeeded) {
			console.log("Quest completed!")
			FluxDispatcher.unsubscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
		}
	}
	FluxDispatcher.subscribe("QUESTS_SEND_HEARTBEAT_SUCCESS", fn)
	
	console.log(`Spoofed your stream to ${quest.config.applicationName}. Stay in vc for ${Math.ceil(quest.config.streamDurationRequirementMinutes - (quest.userStatus?.streamProgressSeconds ?? 0) / 60)} more minutes.`)
	console.log("Remember that you need at least 1 other person to be in the vc!")
}
```
7. Keep the stream running for 15 minutes
8. You can now claim the reward in User Settings -> Gift Inventory!

You can track the progress by looking at the `Quest progress:` prints in the Console tab, or by reopening the Gift Inventory tab in settings. The progress should update every 30s.

## FAQ

**Q: Ctrl + Shift + I doesn't work**

A: Either download the [ptb client](https://discord.com/api/downloads/distributions/app/installers/latest?channel=ptb&platform=win&arch=x64), or use [this](https://www.reddit.com/r/discordapp/comments/sc61n3/comment/hu4fw5x/) to enable DevTools on stable


**Q: I get an error saying "Unauthorized"**

A: Discord has patched the script from working in browsers. Use the desktop app, or alternatively find some extension which lets you change your User-Agent and append the string `Electron/` anywhere in it.

They have also started checking how many people are in the vc, so make sure you join it on at least 1 other account.


**Q: I get a different error**

A: Make sure you've started streaming *before* running the script. Also make sure you're copy/pasting it correctly.
