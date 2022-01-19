<main>
	<div class="wrapper">
		<div class="important">
			<a class="project" href="/graphgen">Graph Generator</a>
		</div>
		<div class="normal">
		</div>
		<div class="normal">
		</div>
		<div class="normal">
		</div>
	</div>
</main>
<style>
	.project{
		height: 100%;
		width: 100%;
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
		font-size:xx-large;
		margin-bottom: 0px;
		color: white;
	}
	.project:hover{
		border-radius: 0rem;
		color:black;
	}
	.wrapper{
		display: grid;
		grid-template-columns: 50% 50%;
		grid-auto-rows: 50vh;
		grid-gap:0;
		margin:0;
	}
	.wrapper > div{
		text-align: center;
		justify-content: center;
		align-items: center;
	}
	.wrapper > div:nth-child(4n+0) {
		background-color: #444444;
	}
	.wrapper > div:nth-child(4n+1){
		background-color: #444444;
	}
	.wrapper > div:nth-child(4n+2){
		background-color: #555555;
	}
	.wrapper > div:nth-child(4n+3) {
		background-color: #555555;
	}
</style>