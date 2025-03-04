<section class="wrapper">
	<div class="title">
	</div>
	<div class="contents">
	</div>
</section>

<style lang="scss">
	.title {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 0;
		margin-bottom: 1.5rem;

		@media (max-width: 868px) {
			align-items: normal;
		}
	}

	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.5rem;
	}

	.contents {
		justify-content: center;
		display: flex;
		flex-wrap: wrap;
		gap: 2rem;
		margin-bottom: 3rem;

		@media (max-width: 868px) {
			justify-content: left;
		}
	}

	img {
		border-radius: 50%;
		height: 64px;
		width: 64px;
	}
</style>
