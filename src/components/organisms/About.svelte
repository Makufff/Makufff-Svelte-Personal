<script lang="ts">
	import RichPresence from '../molecules/RichPresence.svelte';
	import Tooltip from '../atoms/Tooltip.svelte';
	import { fade, slide } from 'svelte/transition';

	let showChatField = false;
	let isTyping = false;

	let getAge = () => {
		let birthDate = new Date('2005/05/27');
		const ageMs = Date.now() - birthDate.getTime();
		const preciseAge = (ageMs / 31536000000).toFixed(10);
		return preciseAge;
	};

	let age = getAge();
	setInterval(() => {
		age = getAge();
	}, 1000);

	let userMessage = '';
	let messages: { type: 'user' | 'bot', text: string }[] = [];

	function toggleChatField() {
		showChatField = !showChatField;
	}

	function sendMessage() {
		if (userMessage.trim() !== '') {

			messages = [...messages, { type: 'user', text: userMessage }];
			
			isTyping = true;
			
			setTimeout(() => {

				isTyping = false;
			const botResponse = "Ha Ha Ha 🤡";
			messages = [...messages, { type: 'bot', text: botResponse }];
					
			setTimeout(scrollToBottom, 100);

			userMessage = '';
			}, 1000);
		}
	}

	function scrollToBottom() {
		const messageContainer = document.querySelector(".message-container");
		if (messageContainer) {
			messageContainer.scrollTop = messageContainer.scrollHeight;
	}
	}

	const quickChatOptions = [
		{ label: 'Hello', message: 'Hi there!' },
		{ label: 'Who are you?', message: 'I am Makufff\'s personal chatbot 🤖' },
		{ label: 'Fun fact?', message: 'Did you know I love music and tech? 🎧' }
	];

	function handleQuickChat(option: { label: string, message: string }) {
		userMessage = option.message;
		sendMessage();
	}
</script>

<section id="about" class="wrapper">
	<div>
		<RichPresence />
	</div>
	<div class="text">
		<h2>bio</h2>
		<p>
			Hey there! I'm Makufff, <Tooltip tip={age}><span>{Math.floor(Number(age))}</span></Tooltip> years old
			<br>I love Rock, Jazz, and Lofi, with artists like
			<Tooltip tip="Blank Space">
				<a href="https://youtu.be/e-ORhEE9VVg?si=w9jXQsLNb1rdEDlT" target="_blank" rel="noreferrer">
					<span>Taylor Swift</span>
				</a>
			</Tooltip> and <Tooltip tip="Call Me Maybe">
				<a href="https://youtu.be/fWNaR-rxAic?si=60jz72EJAAJPHNhV" target="_blank" rel="noreferrer">
					<span>Carly Rae Jepsen</span>
				</a>
			</Tooltip>. 🎧
			Right now, I'm studying Rust🦀 and NixOS, and I'm into AI Engineering and Development.
			Music and tech inspire me! 🐈‍⬛ <br> 少しでも会えたら嬉しいな. &#60;3
		</p>
	</div>
</section>

<div 
	class="chatbot-icon" 
	on:click={toggleChatField} 
	role="button" 
	tabindex="0"
	background-color="var(--primanry)"
	on:keydown={(e) => e.key === 'Enter' || e.key === ' ' ? toggleChatField() : null}
	transition:fade
>
🤖
</div>

{#if showChatField}
	<div class="chatbot-field" transition:slide>
		<div class="message-container">
			{#each messages as message}
				<div 
					class="message" 
					class:user-message={message.type === 'user'}
					class:bot-message={message.type === 'bot'}
				>
					{message.text}
				</div>
			{/each}
			{#if isTyping}
				<div class="message bot-message typing">
					<span class="typing-dots">...</span>
				</div>
			{/if}
		</div>

		<div class="quick-chat-options">
			{#each quickChatOptions as option}
				<button class="quick-chat-btn" on:click={() => handleQuickChat(option)}>
					{option.label}
				</button>
			{/each}
		</div>

		<div class="input-area">
			<textarea 
				bind:value={userMessage} 
				placeholder="Type your message..." 
				rows="3"
				on:keydown={(e) => e.key === 'Enter' && !e.shiftKey ? (e.preventDefault(), sendMessage()) : null}
			></textarea>
			<button on:click={sendMessage} disabled={userMessage.trim() === ''}>
				Send
			</button>
		</div>
	</div>
{/if}

<style lang="scss">
	section {
		margin-bottom: 6rem;
		display: grid;
		gap: 4.5rem;
		grid-template-columns: 1fr 1fr;
		align-items: center;
	}

	.text {
		position: relative;
		line-height: 1.75rem;
	}

	span {
		font-weight: 400;
		font-family: var(--font-two);
		font-size: 0.9rem;
		background-color: var(--elevation-one);
		border-radius: 7px;
		color: var(--accent);
		padding: 0.2rem 0.5rem 0.2rem;
		width: fit-content;
	}

	a {
		text-decoration: none;
	}

	.text::before {
		content: 'DAMM';
		position: absolute;
		font-size: 300px;
		color: rgba(0,0,0,0.12);
		transform: translate(60%, -5%);
		z-index: -1;
	}

	h2 {
		display: none;
		margin-top: 1rem;
	}

	@media (max-width: 868px) {
		section {
			display: flex;
			flex-direction: column;
			align-items: normal;
		}

		h2 {
			display: block;
			margin-bottom: 1rem;
		}
	}

	.chatbot-icon {
		position: fixed;
		bottom: 20px;
		right: 20px;
		background-color: var(--elevation-one);
		border-radius: 50%;
		width: 50px;
		height: 50px;
		display: flex;
		justify-content: center;
		align-items: center;
		cursor: pointer;
		box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
		transition: transform 0.3s ease, box-shadow 0.3s ease;
		z-index: 1000;
	}

	.chatbot-icon:hover {
		transform: scale(1.1);
		box-shadow: 0 6px 15px rgba(0, 0, 0, 0.2);
	}

	.chatbot-field {
		position: fixed;
		bottom: 100px;
		right: 20px;
		background-color: var(--elevation-one);
		border: 1px solid var(--elevation-three);
		border-radius: 12px;
		box-shadow: 0 6px 15px rgba(0, 0, 0, 0.1);
		width: 350px;
		display: flex;
		flex-direction: column;
		gap: 15px;
		z-index: 1000;
		max-height: 500px;
	}

	.message-container {
		height: 300px;
		overflow-y: auto;
		padding: 15px;
		display: flex;
		flex-direction: column;
		gap: 10px;
		scrollbar-width: thin;
		scrollbar-color: var(--elevation-three) transparent;
	}

	.message {
		max-width: 80%;
		padding: 10px;
		border-radius: 10px;
		word-wrap: break-word;
		clear: both;
	}

	.user-message {
		background-color: var(--accent);
		color: white;
		align-self: flex-end;
		margin-left: auto;
	}

	.bot-message {
		background-color: var(--accent-opacity);
		color: var(--text-primary);
		align-self: flex-start;
	}

	.typing {
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.typing-dots {
		animation: typing 1.4s infinite;
		font-size: 20px;
	}

	@keyframes typing {
		0%, 100% { opacity: 0.2; }
		50% { opacity: 1; }
	}

	.quick-chat-options {
		display: flex;
		justify-content: center;
		gap: 10px;
		padding: 0 15px;
	}

	.quick-chat-btn {
		background-color: var(--elevation-two);
		color: var(--text-secondary);
		border: none;
		padding: 5px 10px;
		border-radius: 20px;
		font-size: 0.8rem;
		cursor: pointer;
		transition: background-color 0.3s;

		&:hover {
			background-color: var(--elevation-three);
		}
	}

	.input-area {
		display: flex;
		gap: 10px;
		padding: 15px;
	}

	.input-area textarea {
		flex-grow: 1;
		border: 1px solid var(--elevation-three);
		border-radius: 8px;
		padding: 10px;
		resize: none;
	}

	.input-area button {
		background-color: var(--accent);
		color: white;
		border: none;
		border-radius: 8px;
		padding: 10px 15px;
		cursor: pointer;
		transition: background-color 0.3s;

		&:disabled {
			opacity: 0.5;
			cursor: not-allowed;
		}

		&:not(:disabled):hover {
			background-color: var(--text-primary);
		}
	}
</style>