<!-- Developed by Taipei Urban Intelligence Center 2023-2024-->
<script setup>
import { onMounted, ref, computed } from "vue";
import { useAdminStore } from "../../store/adminStore";
import http from "../../router/axios";

const adminStore = useAdminStore();

const searchParams = ref({
	searchbyindex: "",
	searchbyname: "",
	sort: "",
	order: "",
	pagesize: 10,
	pagenum: 1,
});
onMounted(() => {
	adminStore.getPublicComponents(searchParams.value);
});
const index = ref("");
const types = ref("");
const unit = ref("");
const name = ref("");
const color = ref("");
async function submitComponentCharToDB() {
	if (!index.value.trim()) return;
	if (!types.value.trim()) return;
	if (!unit.value.trim()) return;
	if (!name.value.trim()) return;
	color.value = "[" + color.value + "]";
	color.value = color.value.replace("[", '["');
	color.value = color.value.replace("]", '"]');
	color.value = color.value.replace(",", '","');
	types.value = "[" + types.value + "]";
	types.value = types.value.replace("[", '["');
	types.value = types.value.replace("]", '"]');
	types.value = types.value.replace(",", '","');
	var time = new Date();
	const data = {};
	const divs = document.querySelectorAll("div");
	data["INDEX"] = index.value;
	data["CREATED_AT"] = time;
	data["UPDATED_AT"] = time;

	divs.forEach((div) => {
		const label = div
			.querySelector("span")
			?.textContent?.trim()
			.toLowerCase();
		const input = div.querySelector("input");
		if (label && input) {
			let value = input.value.trim();
			switch (label) {
				case "map_config_ids":
					// 轉成整數陣列
					data["MAP_CONFIG_IDS"] = value
						? value.split(",").map(Number)
						: [];
					break;
				case "update_freq":
					data["UPDATE_FREQ"] = value ? parseInt(value) : 0;
					break;
				case "links":
					data["LINK"] = value ? value.split(",") : [];
					break;
				case "contributor":
					data["CONTRIBUTOR"] = value ? value.split(",") : [];
					break;
				default:
					data[label.toUpperCase()] = value;
					break;
			}
		}
	});
	delete data["FULLSCREEN"];
	delete data["KEYBOARD_DOUBLE_ARROW_LEFT"];
	const json = JSON.stringify(data, null, 2);
	console.log(json);
	try {
		const res = http.post(
			"http://localhost:8088/api/v1/user/query_charts",
			json
		);
		alert("送出成功！");
	} catch (err) {
		console.error("送出失敗：", err);
		alert("發送失敗，請稍後再試");
	}
	try {
		const res = await http.post(
			"http://localhost:8088/api/v1/user/component_charts",
			{
				INDEX: index.value, //component id
				COLOR: JSON.parse(color.value.split(",")), //user id
				TYPES: JSON.parse(types.value.split(",")), //user name
				UNIT: unit.value, //comment
			}
		);
		const abc = await http.post(
			"http://localhost:8088/api/v1/user/components",
			{
				INDEX: index.value, //component id
				NAME: name.value,
			}
		);
		alert("送出成功！");
	} catch (err) {
		console.error("送出失敗：", err);
		alert("發送失敗，請稍後再試");
	}
}
const submes = ref("");
const returnmes = ref("");
const wait = ref("");
async function getAIreturn() {
	wait.value = "Waiting for AI!";
	const response = await fetch("http://localhost:11434/api/generate", {
		method: "POST",
		headers: {
			"Content-Type": "application/json",
		},
		body: JSON.stringify({
			model: "deepseek-coder:6.7b",
			prompt: submes.value,
			stream: false,
		}),
	});
	wait.value = "";
	returnmes.value = await response.json();
	console.log(returnmes.value.response); // data.response 包含模型回應的文字
}
</script>

<template>
	<div style="font-size: 30px">component_charts & component</div>
	<table style="height: 23%">
		<tr>
			<td>資料(.CSV)</td>
			<td>index</td>
			<td>color</td>
			<td>types</td>
			<td>unit</td>
			<td>name</td>
		</tr>
		<tr>
			<td class="file-uploader">
				<input type="file" @change="handleFileChange" />
				<p v-if="selectedFile">已選擇檔案：{{ selectedFile.name }}</p>
			</td>
			<td>
				<input v-model="index" type="text" />
			</td>
			<td>
				<input v-model="color" type="text" />
			</td>
			<td>
				<input v-model="types" type="text" />
			</td>
			<td>
				<input v-model="unit" type="text" />
			</td>
			<td>
				<input v-model="name" type="text" />
			</td>
		</tr>
	</table>
	<div>
		<div style="font-size: 30px">
			query_charts
			&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;
			AI help
		</div>
		<div
			style="
				position: absolute;
				width: 25%;
				height: 55%;
				left: 14%;
				bottom: 3%;
				overflow-y: auto; /* 垂直滾動 */
				padding: 6px;
				border: none;
				background-color: #555;
			"
		>
			<label>
				<!-- <div>
				<span>history_config</span>: 
				<label><input type="text" class="IN" /></label>
			</div> -->
				<!-- <div>
				<span>map_config_ids</span> <input type="text" class="IN" />
			</div> -->
				<!-- <div><span>map_filter</span> <input type="text" class="IN" /></div> -->
				<div>
					<span>time_from</span> <input type="text" class="IN" />
				</div>
				<div><span>time_to</span> <input type="text" class="IN" /></div>
				<!-- <div><span>update_freq</span> <input type="int" class="IN" /></div> -->
				<div>
					<span>update_freq_unit</span>
					<input type="text" class="IN" />
				</div>
				<div><span>source</span> <input type="text" class="IN" /></div>
				<div>
					<span>short_desc</span> <input type="text" class="IN" />
				</div>
				<div>
					<span>long_desc</span> <input type="text" class="IN" />
				</div>
				<div>
					<span>use_case</span> <input type="text" class="IN" />
				</div>
				<div><span>links</span> <input type="text" class="IN" /></div>
				<div>
					<span>contributor</span> <input type="text" class="IN" />
				</div>
				<div>
					<span>query_type</span> <input type="text" class="IN" />
				</div>
				<div>
					<span>query_chart</span> <input type="text" class="IN" />
				</div>
				<div>
					<span>query_history</span> <input type="text" class="IN" />
				</div>
				<div><span>city</span> <input type="text" class="IN" /></div>
				<button id="upload" @click="submitComponentCharToDB">
					上傳
				</button>
			</label>
		</div>
	</div>
	<div
		style="
			width: 50%;
			height: 55%;
			position: absolute;
			right: 5%;
			bottom: 20px;
			padding: 6px;
			overflow-y: auto; /* 垂直滾動 */
			background-color: #555;
		"
	>
		<p style="font-size: 30px">{{ returnmes.response }}{{ wait }}</p>
	</div>
	<input
		v-model="submes"
		type="text"
		style="
			position: absolute;
			right: 5%;
			bottom: 3%;
			height: 30px;
			width: 50%;
		"
	/>
	<button
		@click="getAIreturn"
		style="
			background-color: black;
			position: absolute;
			width: 40px;
			height: 35px;
			bottom: 3%;
			right: 5.5%;
		"
	>
		ask
	</button>
</template>
<style>
label {
	text-align: right;
}
#upload {
	position: relative;
	background-color: gray;
	height: 30px;
	width: 100%;
}
</style>
