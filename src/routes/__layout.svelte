<main>
	<div class="wrapper">
		<div class="navbar">
			<a href="/" class="navbutton">Home</a>
			<a href="/projects" class="navbutton">Projects</a>
		</div>
		<div class="content">
			<slot />
		</div>
	</div>
</main>
<style>
	:global(body){
		margin:0;
	}

	.wrapper{
		display: grid;
		grid-template-columns: 6rem auto;
		grid-template-rows: minmax(100vh,max-content);
		grid-gap:0px;
		background-color: black;
	}
	.navbar{
		background-color: #222222;
	}
	.navbutton{
		height: 4rem;
		width: 6rem;
		position: relative;
		display: flex;
		align-items: center;
		justify-content: center;
		margin: 0;
		text-decoration: none;
		border-radius: 0rem;
		transition-property: background-color, border-color, color, fill, stroke, border-radius;
    	transition-timing-function: linear;
    	transition-duration: 150ms;
		font-size:x-large;
		margin-bottom: 5px;
		color: white;
		background-color: black;
	}
	.navbutton:hover{
		border-radius: 1rem;
		background-color: white;
		color:black;
	}
	.content{
		background-color: gray;
	}
</style>