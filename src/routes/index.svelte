
<div class="wrapper">
	<a href="/planetgen" id="planet-gen">
		<div>
			<img src="./images/earth.jpg" alt="Planet Gen Logo" height=200px>
			<br>
			Planet Generator Beta Here!
		</div>
	</a>
</div>
<style>
	.wrapper{
		display: grid;
		grid-gap: 0px;
		grid-template-columns: repeat(4,25%);
		grid-auto-rows: 25vh;
		align-items: center;
		justify-content: center;
		text-align: center;
	}
	.wrapper>a{
		width: 100%;
		height: 100%;
		align-items: center;
		justify-content: center;
		text-align: center;
		display: flex;
	}
	#planet-gen{
		grid-column: span 4;
		grid-row: span 4;
		background-color: #111111;
		padding: 0;
		margin: 0;
		color:white;
		transition-property: background-color, border-color, color;
    	transition-timing-function: linear;
    	transition-duration: 150ms;
		font-size: 3rem;
		text-decoration: none;
		width: 100%;
		height: 100%;
		align-items: center;
		justify-content: center;
		text-align: center;
		display: flex;
		
	}
	#planet-gen:hover{
		background-color: #555555;
	}
</style>