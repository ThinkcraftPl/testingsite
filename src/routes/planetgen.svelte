<script>
	import * as THREE from 'https://unpkg.com/three/build/three.module.js'
	import {onMount} from 'svelte'
	let scene=null,camera=null,renderer=null,geometry=null,material=null,cube=null;
	onMount(()=>{
		let canvas=document.getElementById('viewport');
		scene = new THREE.Scene();
		camera = new THREE.PerspectiveCamera( 75, viewport.clientWidth/viewport.clientHeight, 0.1, 1000 );
		renderer = new THREE.WebGLRenderer({canvas});
		geometry = new THREE.BoxGeometry();
		material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
		cube = new THREE.Mesh( geometry, material );
		scene.add( cube );
		camera.position.z = 5;

		animate();
	});
	function animate() {
		requestAnimationFrame( animate );
		cube.rotation.x += 0.01;
		cube.rotation.y += 0.01;
		
		renderer.setSize((window.innerWidth-96)*(sidebarHidden?0.99:0.84),window.innerHeight,false);
		const canvas = renderer.domElement;
		camera.aspect = canvas.clientWidth / canvas.clientHeight;
		camera.updateProjectionMatrix();
		renderer.render( scene, camera );
	}

	let sidebarHidden=false;
	function toggleSidebar(){
		sidebarHidden=!sidebarHidden;
		return false;
	}
</script>
<div class="main_wrapper {sidebarHidden===true?'sidebar-hidden':''}">
	<div class="sidebar" style="visibility: {sidebarHidden?"hidden":"visible"}">
	</div>
	<button class="sidebar-button" on:click={toggleSidebar}>
	</button>
	<canvas id="viewport" style="width: 100%; height: 100%; display:block; margin: 0;">
	</canvas>
</div>
<style>
	:global(body){
		
		margin: 0;
		height: 100%;
	}
	.main_wrapper{
		display: grid;
		grid-column-gap:0;
		grid-template-rows: minmax(100vh,max-content);
		grid-template-columns: 15% minmax(1%,20px) auto;
		
		-o-transition: all 0.1s;
		-moz-transition: all 0.1s;
		-webkit-transition: all 0.1s;
		transition: all 0.1s;

		background-color: #333333;
	}
	.main_wrapper.sidebar-hidden{
		grid-template-columns: 0% minmax(1%,20px) auto;
	}
	.sidebar{
		background-color: #333333;
	}
	.sidebar-button{
		display: flex;
		border: 0px solid transparent;
		background-color: #111111;
		color:white;
		justify-content: center;
		align-items: center;
		text-align: center;
		padding: 0;
	}
</style>