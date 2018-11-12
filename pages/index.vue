<template>
	<div class="main">
		<transition name="fade">
		<div class="new" v-if="new_app.open==true">
			<form class="new" v-on:submit.prevent="send_app">
				<label class="close" @click="new_app.open=false">Закрыть</label>
				<label style="font-weight: 600; font-size: 1.25rem">Новая заявка</label>
				<label class="new">Имя клиента</label>
				<input type="text" class="new" v-model="new_app.data.nameorder" required>
				<label class="new">Номер телефона</label>
				<input type="text" class="new" v-model="new_app.data.phone" required>
				<label class="new">Адрес</label>
				<input type="text" class="new" v-model="new_app.data.address" required>
				<label class="new">Вид поломки</label>
				<select class="new" v-model="new_app.data.problemstatus" required>
					<option v-for="item in breakage" :value="item.id">{{item.breakagename}}</option>
				</select>
				<label class="new">Мастер</label>
				<select class="new" v-model="new_app.data.masterid" required>
					<option v-for="item in masters" :value="item.id">{{item.name}}</option>
				</select>
				<input type="submit" class="sub">
			</form>
		</div>
		</transition>
		<div class="left" v-bind:class="{blure: new_app.open}">
			<div class="left_pad" style="background-image: url('/message.png'); background-position: center; background-size: contain;" v-bind:class="{lp_picked: table==0}" @click="table=0" title="Заявки">
			</div>
			<div class="left_pad" style="background-image: url('/people.png'); background-position: center; background-size: contain;" v-bind:class="{lp_picked: table==1}" @click="table=1" title="Мастера">
			</div>
			<div class="left_pad" style="background-image: url('/add.png'); background-position: center; background-size: contain;" @click="new_app.open=true" title="Новая заявка">
			</div>
			<div class="left_pad turn">
			</div>
		</div>
		<div class="right" v-if="table==0" v-bind:class="{blure: new_app.open}">
			<div class="row header">
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
					Поломка
				</div>
				<div class="col">
					Статус заявки
				</div>
			</div>
			<div class='row' v-for="item in app">
				<div class="col">
					{{ item.address }}
				</div>
				<div class="col">
					{{ item.phone }}
				</div>
				<div class="col">
					{{ item.nameorder }}
				</div>
				<div class="col">
					{{ masterName(item.idmaster) }}
				</div>
				<div class="col">
					{{ pName(item.problemstatus) }}
				</div>
				<div class="col">
					{{ sName(item.statusorder) }}
				</div>
			</div>
		</div>
		<div class="right" v-if="table==1">
			<div class="row header">
				<div class="col">
					Имя
				</div>
				<div class="col">
					Телефон
				</div>
				<div class="col">
					Статус
				</div>
			</div>
			<div class='row' v-for="item in masters">
				<div class="col">
					{{ item.name }}
				</div>
				<div class="col">
					{{ item.phone }}
				</div>
				<div class="col" v-if="item.status==0 && item.acceptance==1">
					Занят
				</div>
				<div class="col" v-else-if="item.status==1 && item.acceptance==1">
					Свободен
				</div>
				<div class="col" v-else-if="item.acceptance==0">
					Не принят
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	import axios from 'axios';
	export default {
		async asyncData(){
			async function select(item){
				let query = await axios.post(`https://asterisk.svo.kz/dom/getinf`);
				if(item=='app'){
					return query.data.app;
				} else if(item=='app_status'){
					return query.data.app_status;
				} else if(item=='masters'){
					return query.data.masters;
				} else if(item=='breakage'){
					return query.data.breakage;
				}
				
			}
			return {
				table: 0,
				app: await select('app'),
				masters: await select('masters'),
				app_status: await select('app_status'),
				breakage: await select('breakage'),
				new_app: {
					open: false,
					data: {}
				}
			}

		},
		methods: {
			masterName(item){
				for(var i=0; i<this.masters.length; i++){
					if(item==this.masters[i].id){
						return this.masters[i].name;
					}
				}
			},
			sName(item){
				var result = 0;
				for(var i=0; i<this.app_status.length; i++){
					if(item==this.app_status[i].id){
						result = this.app_status[i].name;
					}
				}
				return result
			},
			pName(item){
				console.log(this.breakage)
				var result = 0;
				for(var i=0; i<this.breakage.length; i++){
					if(item==this.breakage[i].id){
						result = this.breakage[i].breakagename;
					}
				}
				return result
			},
			async send_app(){
				try{
					var query = await axios.post('https://asterisk.svo.kz/dom/new_order', this.new_app.data);
					if(query.status==200){
						this.new_app.data = {};
						this.new_app.open=false;
					}
				} catch(e){
					alert('Ошибка')
				}
								
			},
			wss(){
				var self = this;
				var socket = new WebSocket("ws://asterisk.svo.kz/8001");
				socket.onmessage = function(event){
					var data = JSON.parse(event.data);
					var url = data.action;
					switch(url){
						case 'app':
							self.app = [];
							for(var i=0; i<data.data.length; i++){
								self.app.unshift(data.data[i]);
							}
							break;
						case 'driver':
							self.driver = [];
							for(var i=0; i<data.data.length; i++){
								self.driver.unshift(data.data[i]);								
							}
							break;
						case 'new_app':
							var app = data.data;
							var check = true;
							for(var i=0; i<self.app.length; i++){
								if(self.app[i].id == app.id){
									self.app.splice(i, 1);
									self.app.unshift(app);
									check = false;
									break;
								} else {
									check = true;
								}
							}
							if(check){
								self.app.unshift(app);
							}
							var audio = new Audio();
							audio.src='/message.mp3';
							audio.play();
							break;
						case 'new_driver':
							var driver = data.data;
							var check = true;
							for(var i=0; i<self.driver.length; i++){
								if(self.driver[i].id == driver.id){
									self.driver.splice(i, 1);
									self.driver.unshift(driver);
									check = false;
									break;
								} else {
									check = true;
								}
							}
							if(check){
								self.driver.unshift(driver);
							}
							var audio = new Audio();
							audio.src='/message.mp3';
							audio.play();
							break;
					}
				}
			}
		},
		mounted(){

		}
	}
</script>

<style>
	@import url('https://fonts.googleapis.com/css?family=Noto+Serif');

	.fade-enter-active, .fade-leave-active {
		transition: opacity .5s;
	}
	.fade-enter, .fade-leave-to /* .fade-leave-active до версии 2.1.8 */ {
		opacity: 0;
	}

	label.close {
		position: absolute;
		right: 5px;
		top: 5px;
		text-decoration: underline;
		font-size: 0.75rem;
		color: rgba(210, 210, 210, 0.7);
		transition: 0.5s;
	}

	label.close:hover {
		color: red;
		cursor: pointer;
	}

	div.new {
		position: absolute;
		min-height: 100vh;
		min-width: 100vw;
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 999;
		background-color: rgba(0,0,0,0.5);
	}

	form.new {
		position: relative;
		min-width: 400px;
		width: 35%;
		min-height: 400px;
		height: 70vh;
		background-color: rgb(240, 240, 255);
		border-style: solid;
		border-width: 1px;
		border-color: rgb(102, 179, 255);
		color: rgba(0, 99, 153, 0.7);
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	label.new {
		margin: 1vh;
	}

	input.new, select.new {
		width: 50%;
	}

	.sub {
		margin: 3vh;
		width: 50%;
		height: 8vh;
		background-color: rgb(250, 250, 255);
		color:  rgba(0, 99, 153, 0.7);
		border-color: rgb(102, 179, 255);
		font-family: 'Noto Serif', serif;
	}

	.sub:hover {
		background-color: rgb(210, 210, 255);
		color: white;
		cursor: pointer;
	}

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
		min-width: 50px;
		width: 3rem;
		min-height: 50px;
		height: 3rem;
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

	div.lp_picked {
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

	div.right::-webkit-scrollbar {
    	width: 5px;
   	}

	div.right::-webkit-scrollbar-track {
	    box-shadow: inset 0 0 5px grey; 
	    border-radius: 10px;
	}

/* Handle */
	div.right::-webkit-scrollbar-thumb {
	    background: rgba(0, 99, 153, 0.4); 
	    border-radius: 10px;
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
		font-size: 0.85rem;
		font-weight: 200
	}
	div.row:hover {
		background-color: rgb(240, 240, 255);
		cursor: pointer;
	}

	div.header {
		background-color: rgba(255, 255, 255, 0.9);
		font-size: 1.05rem;
		font-weight: 600;
	}

	div.header:hover {
		background-color: white;
		cursor: initial;
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