<template>
	<div class="main">
		<div class="left">
			<div class="left_pad" style="background-image: url('/message.png'); background-position: center; background-size: contain;">
			</div>
			<div class="left_pad" style="background-image: url('/people.png'); background-position: center; background-size: contain;">
			</div>
			<div class="left_pad turn">
			</div>
		</div>
		<div class="right">
			<div class="row" @click="select()">
				<div class="col">
					Адрес
				</div>
				<div class="col">
					Телефон
				</div>
				<div class="col">
					Имя клиента
				</div>
				<div class="col">
					Имя мастера
				</div>
				<div class="col">
					Статус
				</div>
			</div>
			<div class='row' v-for="item in app">
				<div class="col">
					{{ item.address }}
				</div>
				<div class="col">
					{{ item.address }}
				</div>
				<div class="col">
					{{ item.address }}
				</div>
				<div class="col">
					{{ item.address }}
				</div>
				<div class="col">
					{{ item.address }}
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	import axios from 'axios';
	export default {
		async asyncData(){
			return {
				app: 0
			}

		},
		methods: {
			async select(){
				try{
					let query = await axios.post('https://asteriks.svo.kz/dom/get/inf');
					console.log(query);
				} catch(e){
					console.log(e);
				}				
			}
		},
		mounted(){

		}
	}
</script>

<style>
	@import url('https://fonts.googleapis.com/css?family=Noto+Serif');

	html, body {
		margin: 0;
		padding: 0;
	}
	.main {
		display: flex;
		align-items: center;
		min-height: 100vh;
		font-family: 'Noto Serif', serif;
	}
	div.left {
		display: flex;
		align-items: center;
		min-width: 50px;
		width: 5vw;
		height: 100vh;
		min-height: 600px;
		background-color: rgb(51, 51, 51);
		border-right-style: solid;
		border-width: 1px;
		border-color: rgb(102, 179, 255);
		flex-direction: column;
	}
	div.left_pad {
		display: flex;
		width: 80%;
		min-height: 50px;
		height: 8%;
		color: rgb(230, 230, 230);
		transition: 1s;
		margin-top: 10px;
		text-align: center;
		align-items: center;
		justify-content: center;
		border-radius: 50%;
	}
	div.left_pad:hover {
		cursor: pointer;
		box-shadow: 0 0 100px rgba(102, 179, 255, 0.8);
		color: rgba(102, 179, 255, 1);
		background-color: white;
	}
	div.turn {
		background-image: url('/turn.png');
		background-position: center;
		background-size: contain;
		position: absolute;
		bottom: 10px;
		width: 50px;
		height: 50px;
	}
	div.turn:hover {
		background-image: url('/turn_hover.png');
	}

	div.right {
		display: flex;
		min-width: 750px;
		width: calc(95vw-1px);
		min-height: 600px;
		height: 100vh;
		max-height: 100vh;
		background-color: rgb(230, 230, 255);
		flex-direction: column;
		overflow-y: auto;
	}

	div.row {
		display: flex;
		min-height: calc(600px/10);
		height: 10vh;
		max-height: 10vh;
		min-width: 750px;
		width: 100%;
		border-bottom-style: solid;
		border-color: white;
		border-width: 0.1px;
		justify-content: center;
		align-items: center;
		color: rgba(0, 99, 153, 0.7);
		font-size: 1rem;
		font-weight: 600
	}
	div.row:hover {
		background-color: rgb(240, 240, 255);
		cursor: pointer;
	}

	div.col {
		display: flex;
		justify-content: center;
		align-items: center;
		width: 100%;
		min-height: 10vh;
		border-right-style: solid;
		border-color: white;
		border-width: 0.1px;
	}
</style>