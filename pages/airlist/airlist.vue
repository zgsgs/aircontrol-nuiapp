<template>
	<view class="container">
		<view class="content">
			<checkbox-group class="ul" @change="checkboxChange">
				<view class="li" v-for="(item, index) in airlist" :key="index">
					<label class="label" :for="'cb' + index">
						<checkbox :id="'cb' + index" :value="item.name" :checked="item.checked" />
						<view class="device-title">{{ item.name }}</view>
					</label>
					<view class="btn-set">
						<!-- <button class="btn del" @click="del(item.key)">删除</button> -->
						<button class="btn save" @click="use(item.deviceName)">使用</button>
					</view>
				</view>
			</checkbox-group>

			<!-- 	<ul id="device-list" v-for="(item, index) in airlist" :key="index">
				<li>
					<checkbox name="deviceSet" value="${arr[index].DeviceName}" id="dev${index}" />
					<view class="checkbox">
						<label for="dev${index}">
							<view class="device-title">{{ item.deviceName }}</view>
						</label>
					</view>
					<view class="btn-set">
						<button class="btn"></button>
						<button class="btn del" @click="del(item.key)">删除</button>
						<button class="btn save" @click="use(item.deviceName)">使用</button>
					</view>
				</li>
			</ul> -->
			<!-- <view class="btn-set">
			<button class="btn save" @click="photoScanCode">允许从相机和相册扫码</button>
			<button class="btn save" @click="inItMqtt">初始化mqtt</button>
		</view> -->
		</view>
	</view>
</template>

<script>
import * as crypto from 'crypto';
import * as mqtt from 'alibabacloud-iot-device-sdk';
import Utils from '../../common/js/utils.js';
export default {
	data() {
		return {
			airlist: [],
			checkedList: []
		};
	},
	onNavigationBarButtonTap() {
		let that = this;
		console.log('点击了自定义按钮');
		uni.setStorageSync('$checkedDeviceList', JSON.stringify(that.checkedList))
		Utils.goTo('aircontrol')
	},
	onLoad() {
		this.createList();
	},
	methods: {
		createList: function() {
			let that = this;
			let arr = uni.getStorageSync('$DeviceList') || '[]';
			arr = JSON.parse(arr);
			arr = this.checkBox(arr);
			arr = Utils.unique(arr, 'deviceName')
			console.log(arr)
			this.airlist = arr
		},
		checkBox: function(arr) {
			if (arr) {
				let list = [];
				arr.forEach(obj => {
					let item = {};
					item.name = obj.deviceName;
					item.checked = false;
					list.push(item);
				});
				return list;
			}
		},
		checkboxChange: function(e) {
			this.checkedList = e.detail.value
			console.log(e.detail.value)
		},
		photoScanCode() {
			// 允许从相机和相册扫码
			uni.scanCode({
				success: function(res) {
					console.log('条码类型：' + res.scanType);
					console.log('条码内容：' + res.result);
					this.airlist.push(res.result);
				}
			});
		},
		use: function(val) {
			if (val) {
				this.checkArr.push(val);
				uni.setStorageSync('$UseDeviceList', JSON.stringify(this.checkArr));
			}
			Utils.goTo('aircontrol');
		},

		//测试代码
		inItMqtt() {
			let that = this;
			console.log(11);
			let deviceConfig = {
				productKey: 'a1ziFZuhXoc',
				deviceName: '26_test',
				deviceSecret: 'yXVlDFCGS8ImnaUzGYJ5LPbsl1RtGv1t',
				topic: 'aircontrol',
				regionId: 'cn-shanghai'
			};
			const options = {
				productKey: deviceConfig.productKey,
				deviceName: deviceConfig.deviceName,
				timestamp: Date.now(),
				clientId: Math.random()
					.toString(36)
					.substr(2)
			};
			console.log(options);
			options.password = that.signHmacSha1(options, deviceConfig.deviceSecret);
			console.log(options.password);
			options.clientId = `${options.clientId}|securemode=3,signmethod=hmacsha1,timestamp=${options.timestamp}|`;
			options.username = `${options.deviceName}&${options.productKey}`;

			const url = `tcp://${deviceConfig.productKey}.iot-as-mqtt.${deviceConfig.regionId}.aliyuncs.com:1883`;
			console.log(url);
			//建立连接
			const client = mqtt.connect(
				url,
				options
			);
			console.log(client);
			//属性上报的Topic
			const topic = `/sys/${deviceConfig.productKey}/${deviceConfig.deviceName}/thing/event/property/post`;
			//发布数据到topic
			client.publish(topic, that.getPostData());
		},

		getPostData: function() {
			const payloadJson = {
				id: Date.now(),
				params: {
					temperature: Math.floor(Math.random() * 20 + 10),
					humidity: Math.floor(Math.random() * 40 + 60)
				},
				method: 'thing.event.property.post'
			};

			// console.log('===postData topic=' + topic);
			console.log(payloadJson);

			return JSON.stringify(payloadJson);
		},

		/*
		  生成基于HmacSha1的password
		  参考文档：https://help.aliyun.com/document_detail/73742.html?#h2-url-1
		*/
		signHmacSha1: function(options, deviceSecret) {
			let keys = Object.keys(options).sort();
			// 按字典序排序
			keys = keys.sort();
			const list = [];
			keys.map(key => {
				list.push(`${key}${options[key]}`);
			});
			const contentStr = list.join('');
			console.log(contentStr);
			return crypto
				.createHmac('sha1', deviceSecret)
				.update(contentStr)
				.digest('hex');
		}
	}
};
</script>

<style>
@import url('@/common/css/reset.css');
@import url('@/common/css/airlist.css');
</style>
