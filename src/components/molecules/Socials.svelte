<script>
	import Social from '../atoms/Social.svelte';
	import { user } from '../../util/discord';

	import { GitHubIcon } from '@indaco/svelte-iconoir/github';
	import { DiscordIcon } from '@indaco/svelte-iconoir/discord';
	import { MailIcon } from '@indaco/svelte-iconoir/mail';
	import { LinkedInIcon } from '@indaco/svelte-iconoir/linkedin';
</script>

<div class="socials-container">
	<Social tip="Makufff" link="https://github.com/makufff">
		<GitHubIcon color="var(--accent)" />
	</Social>

	<Social tip={user.username} link={`https://discord.com/users/${user.id}`}>
		<DiscordIcon color="var(--accent)" />
	</Social>

	<Social tip="makufff.tanapat@gmail.com" link="mailto:makufff.tanapat@gmail.com">
		<MailIcon color="var(--accent)" />
	</Social>

	<Social tip="Tanapat Chamted" link="https://www.linkedin.com/in/tanapat-chamted-124686284/">
		<LinkedInIcon color="var(--accent)" />
	</Social>
</div>

<style>
	.socials-container {
		display: flex;
		gap: 0.75rem;
	}
</style>
