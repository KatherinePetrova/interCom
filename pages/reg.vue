<template>
	<div class="main">
		<form v-on:submit.prevent="reg">
			<label style="position: absolute; top: 10%">Регистрация</label>
			<label class="po">Логин</label>
			<input type="text" class="text" required v-model="user.login" id="login" v-on:change="compare">
			<label class="po">Пароль</label>
			<input type="password" class="text" required v-model="user.pass">
			<label class="po">Почта</label>
			<input type="email" class="text" required v-model="user.email">
			<div style="margin-top: 10px; font-size: 1rem">{{ warning }}</div>
			<input type="submit" class="submit" id="submit" v-bind:disabled="user.dis">
			<nuxt-link to="/" class="reg">Авторизация</nuxt-link>
		</form>
	</div>
</template>

<script>
import axios from 'axios'
export default {
	data(){
		return {
			user: {
				dis: false
			},
			warning: ''
		}	
	},
	methods: {
		async compare(){
			try{
				var query = await axios.post('https://asterisk.svo.kz/dom/compare', {login: this.user.login})
				if(query.status == 200){
					document.getElementById('submit').style.backgroundColor = "rgba(255, 102, 102, 0.5)";
					document.getElementById('login').style.backgroundColor = "rgba(255, 102, 102, 0.5)";
					this.user.dis = true;
				}
			} catch(e){
				document.getElementById('login').style.backgroundColor = "rgba(51, 255, 173, 0.5)";
				document.getElementById('submit').style.backgroundColor = "rgba(51, 255, 173, 0.5)";
				this.user.dis = false;
			}
		},
		async reg(){
			var query = await axios.post('https://asterisk.svo.kz/dom/new', this.user);
			if(query.status == 200){
				this.$router.push('/')
			} else {
				this.warning = query.message;
			}
		}
	},
	mounted(){

	}
}
</script>

<style>
	html, body {
		margin: 0;
		padding: 0;
	}
</style>

<style scoped>
	@import url('https://fonts.googleapis.com/css?family=Noto+Serif');
	.main {
		min-height: 100vh;
		min-width: 100vw;
		position: absolute;
		display: flex;
		justify-content: center;
		align-items: center;
		background-color: white;
		font-family: 'Noto Serif', serif;
	}
	form {
		display: flex;
		position: relative;
		background-color: rgb(230, 230, 255);
		box-shadow: 0 0 60px rgba(102, 179, 255, 0.8);
		border-style: solid;
		border-width: 1px;
		border-color: rgba(102, 179, 255, 0.8);
		min-height: 400px;
		min-width: 450px;
		width: 40vw;
		height: 40vw;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		color:  rgba(0, 99, 153, 1);
		font-size: 1.5rem;
		border-radius: 50%;
	}

	label.po {
		padding: 10px;
		font-size: 1.25rem;
	}

	input.submit {
		margin-top: 20px;
		width: 40%;
		height: 10%;
		background-color: rgba(102, 179, 255, 0.5);
		color: white;
		transition: 0.5s;
	}

	input.submit:hover {
		cursor: pointer;
		background: rgba(102, 179, 255, 1)
	}

	input.text {
		width: 40%;
	}

	.reg:link, .reg:visited {
		font-size: 1rem; 
		margin-top: 10px; 
		color: rgba(51, 51, 51, 0.8);
	}

	.reg:hover, .reg:active {
		font-size: 1rem; 
		margin-top: 10px; 
		color: rgba(0, 99, 153, 1);
	}
</style>