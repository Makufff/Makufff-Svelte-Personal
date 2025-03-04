<script>
	import HeroImage from '../../components/atoms/HeroImage.svelte';
	import Button from '../atoms/Button.svelte';
	import Tooltip from '../atoms/Tooltip.svelte';
	import Socials from '../molecules/Socials.svelte';

	function handleClick() {
		const el = document.querySelector('#aw');
		if (!el) return;
		el.scrollIntoView(true);
	}
</script>

<section id="home" class="wrapper">
	<div>
		<h1>Makufff</h1> 
		<div>
			<h4>AI Engineer & Full Stack Developer</h4>
			<h5>King Mongkut's Inst<Tooltip tip="Information Technology (IT22)">
					<h5><span>[IT]</span></h5>
				</Tooltip>ute of Technology Ladkrabang</h5>
		</div>
		<div class="socials">
			<Socials />
		</div>
		<Button side on:click={handleClick} on:keypress={handleClick}>Discover my work ↓</Button>
	</div>
	<HeroImage />
</section>

<style lang="scss">
	@import '../../styles/mixins.scss';

	section {
		scroll-margin-top: 20rem;
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 5.75rem;
		margin-top: 7rem;

		@media screen and (max-width: 868px) {
			margin-top: 4rem;
		}

		@media screen and (max-width: 768px) {
			margin-bottom: 2.75rem;
		}

		@media screen and (max-width: 500px) {
			display: block;
		}
	}

	.socials {
		margin-top: 1rem;
		margin-bottom: 1.7rem;
	}

	h4 {
		margin-top: 1rem;
	}

	h1::before {
		@include outlineText($content: '///', $translateX: -95%, $letterSpacing: -0.1em);
	}

	@media screen and (max-width: 1240px) {
		h1 {
			min-height: 0vw;
			font-size: clamp(4rem, 10vw, 6.25rem);
		}

		h4 {
			min-height: 0vw;
			font-size: clamp(1.4rem, 2.4vw, 1.5rem);
		}
	}

	span {
		font-weight: 400;
		font-family: var(--font-two);
		font-size: 0.9rem;
		background-color: var(--elevation-one);
		border-radius: 7px;
		color: var(--accent);
		width: fit-content;
	}

</style>
