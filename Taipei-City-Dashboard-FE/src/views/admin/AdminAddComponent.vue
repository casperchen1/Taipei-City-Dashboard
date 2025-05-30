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
	try {
		const res = await http.post(
			"http://localhost:8088/api/v1/user/component_charts",
			{
				INDEX: index.value, //component id
				COLOR: Array[color.value], //user id
				TYPED: Array[types.value], //user name
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
</script>

<template>
	<table>
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
	<div>query_charts</div>
	<div
		style="
			position: absolute;
			width: 25%;
			height: 60%;
			bottom: 1%;
			overflow-y: auto; /* 垂直滾動 */
			padding: 6px;
			border: none;
			background-color: #555;
		"
	>
		<label>
			<div>
				<span>history_config</span>:
				<label><input type="text" class="IN" /></label>
			</div>
			<div>
				<span>map_config_ids</span> <input type="text" class="IN" />
			</div>
			<div><span>map_filter</span> <input type="text" class="IN" /></div>
			<div><span>time_from</span> <input type="text" class="IN" /></div>
			<div><span>time_to</span> <input type="text" class="IN" /></div>
			<div><span>update_freq</span> <input type="text" class="IN" /></div>
			<div>
				<span>update_freq_unit</span> <input type="text" class="IN" />
			</div>
			<div><span>source</span> <input type="text" class="IN" /></div>
			<div><span>short_desc</span> <input type="text" class="IN" /></div>
			<div><span>long_desc</span> <input type="text" class="IN" /></div>
			<div><span>use_case</span> <input type="text" class="IN" /></div>
			<div><span>links</span> <input type="text" class="IN" /></div>
			<div><span>contributor</span> <input type="text" class="IN" /></div>
			<div><span>created_at</span> <input type="text" class="IN" /></div>
			<div><span>updated_at</span> <input type="text" class="IN" /></div>
			<div><span>query_type</span> <input type="text" class="IN" /></div>
			<div><span>query_chart</span> <input type="text" class="IN" /></div>
			<div>
				<span>query_history</span> <input type="text" class="IN" />
			</div>
			<div><span>city</span> <input type="text" class="IN" /></div>
		</label>
	</div>
	<button id="upload" @click="submitComponentCharToDB">上傳</button>
</template>
<style>
label {
	text-align: right;
}
#upload {
	background-color: gray;
	height: 30px;
	width: 50px;
}
</style>
