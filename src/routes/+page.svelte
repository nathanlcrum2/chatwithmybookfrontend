
<script lang="ts">
	import bookcover from '$lib/images/book.png';
	import axios from 'axios';
	import bookavatar from '$lib/images/bookavatar.png';
	import { onMount } from 'svelte';
	import { Avatar } from '@skeletonlabs/skeleton';

	// Types
	interface Person {
		id: number;
		host: boolean;
		name: string;
		color: string;
	}
	interface MessageFeed {
		id: number;
		host: boolean;
		name: string;
		timestamp: string;
		message: string;
		color: string;
	}

	let elemChat: HTMLElement;
	let url = 'https://chatwithbookbackend-7t57425ggq-wm.a.run.app/'; //change this for whatever you host this on

	// Navigation List
	const people: Person[] = [
		{ id: 0, host: true, name: 'The Only Path I See', color: 'variant-soft-primary' },
		{ id: 1, host: false, name: 'You', color: 'variant-soft-tertiary' },
	];
	let currentPerson: Person = people[0];

	// Messages
	let messageFeed: MessageFeed[] = [
		{
			id: 0,
			host: true,
			name: people[0].name,
			timestamp: `${getCurrentTimestamp()}`,
			message: 'Welcome! You can ask me anything about the book!',
			color: people[0].color
		},
	];
	let currentMessage = '';

	// For some reason, eslint thinks ScrollBehavior is undefined...
	// eslint-disable-next-line no-undef
	function scrollChatBottom(behavior?: ScrollBehavior): void {
		elemChat.scrollTo({ top: elemChat.scrollHeight, behavior });
	}

	function getCurrentTimestamp(): string {
		return new Date().toLocaleString('en-US', { hour: 'numeric', minute: 'numeric', hour12: true });
	}

	//make this work for user and system
	function addMessage(host: boolean, message: string ): void {
		let person = 1;
		if (host) {
			person = 0;
		}
		const newMessage = {
			id: messageFeed.length,
			host: host,
			name: people[person].name,
			timestamp: `${getCurrentTimestamp()}`,
			message: message,
			color: people[person].color
		};
		// Update the message feed
		messageFeed = [...messageFeed, newMessage];
		// Clear prompt
		currentMessage = '';
		
		// Smooth scroll to bottom
		// Timeout prevents race condition
		setTimeout(() => {
			scrollChatBottom('smooth');
		}, 0);
	}

	function handleInput(host: boolean, message: string): void {
		if (message === ''){
			return;
		}
		addMessage(false, message);
		let hostMessage = 'Sorry, an error occurred. Please try again later.';
		//call api
		axios.get(url + 'query'+ `?query=${message}`)
		.then((response) => {
			hostMessage = response.data.output;
		}).catch((error) => {
			console.log(error.toJSON());
			if (error.toJSON().status === 429) {
				hostMessage = "You have reached your limit of questions. I had to do this so the API doesn't incur heavy costs. \
				Thank you for demoing this and let me know what you think!"
			}
		}).finally(() => {
			//add message to the messagefeed no matter what
			addMessage(true, hostMessage);
		});
	}

	function onPromptKeydown(event: KeyboardEvent): void {
		if (['Enter'].includes(event.code)) {
			event.preventDefault();
			handleInput(false, currentMessage);
		}
	}

	let randomString = 'randomString';
	function generateRandomString(length: number) {
		const charset = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
		let randomString = "";

		for (let i = 0; i < length; i++) {
			const randomIndex = Math.floor(Math.random() * charset.length);
			randomString += charset.charAt(randomIndex);
		}
		return randomString;
	}
	
	// When DOM mounted, scroll to bottom
	// get current url, and generate randomstring to randomize avatar
	onMount(() => {
		scrollChatBottom();
		randomString = generateRandomString(20);
		if (window.location.href.indexOf('localhost') >= 0 || window.location.href.indexOf('127.0.0.1') >= 0) {
			url = 'http://localhost:5000/'; //backend server url
		}
	});
</script>


<div class="flex flex-col h-full items-center">

	<div class="mt-20">
		<p class="text-5xl text-white/80 font-extralight">Chat With My Book</p>
	</div>
	<div class="flex md:flex-row flex-col mt-14 justify-center items-center gap-8 lg:gap-20">
		<a class="w-[60%] md:w-[30%] lg:w-[25%]" target="_blank" href="https://www.amazon.com/Only-Path-See-Problems-Solving/dp/B09RM1F399/ref=sr_1_1?crid=3A47MJDQK720Z&keywords=the+only+path+i+see&qid=1693983729&sprefix=the+only+path+i+se%2Caps%2C206&sr=8-1"><img src={bookcover} alt="The Only Path I See Book Cover"></a>
		<div class="w-full md:w-[60%] lg:w-[50%] xl:w-[40%] px-8">
			<section class="card w-full">
				<div class="grid grid-row-[1fr_auto]">
					<!-- Conversation -->
					<section bind:this={elemChat} class="max-h-[400px] p-4 overflow-y-auto space-y-4">
						{#each messageFeed as bubble}
							{#if bubble.host === true}
								<div class="grid grid-cols-[auto_1fr] gap-2">
									<Avatar src={bookavatar} width="w-12" />
									<div class="card p-4 {bubble.color} rounded-tl-none space-y-2">
										<header class="flex justify-between items-center">
											<p class="font-bold">{bubble.name}</p>
											<small class="opacity-50">{bubble.timestamp}</small>
										</header>
										<p>{bubble.message}</p>
									</div>
								</div>
							{:else}
								<div class="grid grid-cols-[1fr_auto] gap-2">
									<div class="card p-4 rounded-tr-none space-y-2 {bubble.color}">
										<header class="flex justify-between items-center">
											<p class="font-bold">{bubble.name}</p>
											<small class="opacity-50">{bubble.timestamp}</small>
										</header>
										<p>{bubble.message}</p>
									</div>
									<Avatar src="https://api.multiavatar.com/{randomString}.svg" width="w-12" />
								</div>
							{/if}
						{/each}
					</section>
					<!-- Prompt -->
					<section class="border-t border-surface-500/30 p-4">
						<div class="input-group input-group-divider grid-cols-[auto_1fr_auto] rounded-container-token">
							<span class="input-group-shim"></span>
							<textarea
								bind:value={currentMessage}
								class="bg-transparent border-0 ring-0 p-2"
								name="prompt"
								id="prompt"
								placeholder="Ask a question..."
								rows="1"
								on:keydown={onPromptKeydown}
							/>
							<button class="cc py-1 bg-surface-50-900-token text-xl hover:bg-surface-100-800-token {currentMessage ? 'variant-filled-primary' : 'input-group-shim'}" on:click={() => handleInput(false, currentMessage)}>
								<i class="las la-paper-plane"></i>
							</button>
						</div>
					</section>
				</div>
			</section>
			<p class="mt-8 text-sm">I trained a LLM using my book, which allows you to ask questions about the contents of the book and get answers.
				It uses OpenAI which costs money, so I put a limit of 5 questions per visitor. This is just for demo purposes right now.
			</p>
		</div>
		

	</div>
	
	
</div>

<style>
	.cc {
		background-image:
		radial-gradient(at 100% 0%, rgba(var(--color-secondary-500) / 0.33) 0px, transparent 40%);
	}
</style>
