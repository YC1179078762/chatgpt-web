<template>
	<div class="message-box">
		<div class="left-box">
			<div @click="addMessage" class="add-box">
				新建消息
			</div>
			<div v-for="(item, index) in data.messageList" :key="item.id" @click="chooseMessage(item)"
				:class="{ 'message-item': true, 'message-item-active': item.id === data.chatId }">
				<template v-if="!item.edit">
					<span class="message-item-left">{{ item.title }}</span>
					<a-tooltip title="编辑" color="#108ee9">
						<edit-outlined @click="item.edit = true" />
					</a-tooltip>
					<close-circle-outlined @click="closeMessageItem(index)" />
				</template>
				<template v-if="item.edit">
					<a-input class="message-item-left" v-show="item.edit" @blur="item.edit = false"
						v-model:value="item.title"></a-input>
				</template>
			</div>
		</div>
		<div class="right-box">
			<div class="chat-history"></div>
			<div class="send-message-box">
				<a-textarea v-model:value="data.userMessage" placeholder="发送你的问题" />
					<a-button :disabled="!data.userMessage" @click="sendMessage" type="primary">发送</a-button>
			</div>
		</div>
	</div>
</template>
<script lang="ts" setup>
import axios from "axios"
import { EditOutlined, CloseCircleOutlined } from "@ant-design/icons-vue"
let local = localStorage.getItem('message')

const data = reactive({
	userMessage: "",
	messageList: [] as any,
	chatList: [],
	chatId: ""
})
if (local) {
	local = JSON.parse(local)
	data.messageList = local
} else {
	data.messageList = [{
		id: new Date().getTime(),
		title: '新消息',
		chatList: [],
		edit: false,
	}]
}
const addMessage = () => {
	data.messageList.push({
		id: new Date().getTime(),
		title: '新消息',
		chatList: []
	})
}
const closeMessageItem = (index: number) => {
	data.messageList.splice(index, 1)
}
const chooseMessage = (item: any) => {
	data.chatId = item.id
	data.chatList = item.chatList
}
watch(() => data.chatId, () => {
	const index = data.messageList.findIndex((item: any) => (item.id === data.chatId))
	if (index > -1) {
		data.chatList = data.messageList[index].chatList
	}
})
const sendMessage = () => {
	axios.post('/message',{
		message: data.userMessage
	}).then((res:any) =>{
		data.userMessage = ''
		console.log(res)
	}).catch((e) =>{
		console.log(e)
	})
}
</script>
<style lang="less" scoped>
.message-box {
	border: var(--border-in-light);
	border-radius: 20px;
	box-shadow: var(--shadow);
	color: var(--black);
	background-color: var(--white);
	min-width: 600px;
	min-height: 480px;
	max-width: 900px;
	display: flex;
	overflow: hidden;
	box-sizing: border-box;
	width: var(--window-width);
	height: var(--window-height);
}

.left-box {
	top: 0;
	width: var(--sidebar-width);
	box-sizing: border-box;
	padding: 20px;
	// background-color: var(--second);
	display: flex;
	flex-direction: column;
	box-shadow: inset -2px 0 2px 0 rgba(0, 0, 0, .05);

	.add-box {
		width: 100%;
		padding: 4px;
		text-align: center;
		border-radius: 4px;
		margin-bottom: 8px;
		cursor: pointer;
		border: 1px dashed rgb(224, 224, 230);

		&:hover {
			border: 1px dashed #36ad6a;
		}
	}

	.message-item {
		display: flex;
		align-items: center;
		margin-top: 12px;
		width: 100%;
		padding: 12px 16px;
		border-radius: 4px;
		border: 1px solid rgb(224, 224, 230);
		cursor: pointer;

		&-left {
			flex: 1;
		}

		&:hover {
			background-color: rgb(245 245 245);
		}

		.anticon {
			width: 20px;
		}
	}

	.message-item-active {
		border-color: rgb(75 158 95);
		background-color: rgb(245 245 245);
	}
}

.right-box {
	flex: 1;
	display: flex;
	flex-direction: column;
	.send-message-box {
		padding: 12px;
		height: 160px;
		.ant-input {
			border: 4px;
		}
		display: flex;
		.ant-btn {
			// border-color: #409eff;
			// color: #409eff;
		}
		.ant-input {
			height:  100% !important;
			min-height: 100% !important;
			max-height: 100% !important;
		}
	}
	.chat-history {
		flex: 1;
		border-bottom: 1px solid  rgba(0,0,0,.1);
	}
}

@media only screen and (max-width: 600px) {
	.left-box {
		position: absolute;
		left: -100%;
		z-index: 1000;
		height: var(--full-height);
		transition: all .3s ease;
		box-shadow: none;
	}

	.show-left-box {
		left: 0;
	}
}
</style>