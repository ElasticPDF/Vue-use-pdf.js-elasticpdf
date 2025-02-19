<template>
	<div v-if='is_chinese' class='project-title'>
		<img src="elasticpdf-logo.png" alt="" />
		<div>
			<button class='theme-btn btn-outline-info' @click="getPDFData()">获取PDF数据</button>
			<button class='theme-btn btn-outline-help' @click="outputAnnotations()">导出批注</button>
			<button class='theme-btn btn-outline-success' @click="changeFile()">切换文档</button>
			<button class='theme-btn btn-outline-warning' @click="setUser()">切换用户</button>
			<button class='theme-btn btn-outline-info' @click="openOrCloseAnnotatioList()">打开/关闭列表</button>
			<h2>Vue + pdf.js 及 ElasticPDF 部署案例</h2>
			<img src="logo.png" alt="" />
		</div>

	</div>
	<div v-if='!is_chinese' class='project-title'>
		<img src="elasticpdf-logo.png" alt="" />
		<div>
			<button class='theme-btn btn-outline-info' @click="getPDFData()">Get PDF Data</button>
			<button class='theme-btn btn-outline-help' @click="outputAnnotations()">Export Annotations</button>
			<button class='theme-btn btn-outline-success' @click="changeFile()">Change Files</button>
			<button class='theme-btn btn-outline-warning' @click="setUser()">Change User</button>
			<button class='theme-btn btn-outline-info' @click="openOrCloseAnnotatioList()">Open/Close Anno List</button>
			<h2>Vue + pdf.js && ElasticPDF example</h2>
			<img src="logo.png" alt="" />
		</div>
	</div>
	<iframe v-if='is_elasticpdf' @load='initialPDFEditor()' id='elasticpdf-iframe' src="elasticpdf/web/viewer.html"
		frameborder="0" width="100%" height="700px"></iframe>
	<iframe v-if='!is_elasticpdf' @load='initialPDFEditor()' id='elasticpdf-iframe' src="pdfjs-3.2/web/viewer.html"
		frameborder="0" width="100%" height="700px"></iframe>
</template>

<script setup>
	import {
		ref,
		onMounted
	} from 'vue'

	onMounted(() => {
		listenPDFEditorMessage();
	})

	var is_elasticpdf = ref(false);
	var is_chinese = ref(true);
	// if (navigator.language.startsWith('zh')) {
	// 	is_chinese = true;
	// }

	var elasticpdf_viewer = null;

	function initialPDFEditor() {
		// var tips_language = 'en';
		// var lang = navigator.language || navigator.userLanguage;
		// if (lang.startsWith('zh')){
		// 	tips_language = 'zh-cn';
		// }
		elasticpdf_viewer = document.getElementById('elasticpdf-iframe').contentWindow;
		console.log('elasticpdf_viewer', elasticpdf_viewer);
		var pdf_url = "compressed.tracemonkey-pldi-09.pdf";
		if (is_elasticpdf.value == true) {
			pdf_url = "tutorial.pdf";
		}
		elasticpdf_viewer.initialApp({
			'language': 'zh-cn', // GUI language, support Chinese and English 
			'read_only': true, // read only model which user can't annotate
			'show_annotation_list': true, // default show right annotation list
			'pdf_url': pdf_url,
			'member_info': { //member info including id and name
				'id': 'elasticpdf_id',
				'name': 'elasticpdf',
			},
		});
		console.log('pdfurl', elasticpdf_viewer.PDFViewerApplication.baseUrl);
	}

	function listenPDFEditorMessage() {
		window.addEventListener('message', (e) => {
			if (e.data.source != 'elasticpdf') {
				return;
			}

			// 接收pdf数据
			if (e.data.function_name == 'downloadPDF') {
				let file_name = e.data.content['file_name'];
				let pdf_blob = e.data.content['pdf_blob'];
				let pdf_base64 = e.data.content['pdf_base64'];
				console.log('PDF信息', pdf_base64);
				// 如果文档没有被编辑过，则 pdf_base64 仍然是文件名
				// 接收到 pdf 数据，其中 pdf_base64 可以快捷上传到服务器
				postService('upload-pdf-data', {
					'file_name': file_name,
					'file_id': '123ddasfsdffads',
					'file_data': pdf_base64,
				});
			}

			// pdf 批注编辑回调，可以在此处导出批注并传输到服务器
			if (e.data.function_name == 'annotationsModified') {
				// 仅获取 pdf 批注文件，不写入到 pdf 中
				let this_data = elasticpdf_viewer.pdfAnnotation.outputAnnotations();
				let annotation_content = JSON.stringify(this_data['file_annotation']);
				let file_name = this_data['file_name'];
				console.log('批注信息', annotation_content);
				postService('upload-annotation-data', {
					'file_name': file_name,
					'file_id': '123ddasfsdffads',
					'file_annotation': annotation_content,
				});
			}

			// pdf 加载结束的回调，可以在此处导入服务器上储存的批注文件
			if (e.data.function_name == 'pdfLoaded') {
				console.log('PDF加载成功');
				reloadData();
			}
		});
	}

	// 获取pdf数据
	function getPDFData() {
		if (is_elasticpdf.value==false){
			alert('only available for elasticpdf');
			return;
		}
		elasticpdf_viewer.getPDFData();
	}

	//打开或者关闭批注列表
	function openOrCloseAnnotatioList() {
		if (is_elasticpdf.value==false){
			alert('only available for elasticpdf');
			return;
		}
		elasticpdf_viewer.editAnnotation();
	}


	//导出可保存的批注对象
	function outputAnnotations() {
		if (is_elasticpdf.value==false){
			alert('only available for elasticpdf');
			return;
		}
		var this_data = elasticpdf_viewer.pdfAnnotation.outputAnnotations();
		var content = JSON.stringify(this_data['file_annotation']);
		console.log('导出批注', content);
	}

	function changeFile() {
		if (is_elasticpdf.value==false){
			alert('only available for elasticpdf');
			return;
		}
		var test_pdf = 'https://mozilla.github.io/pdf.js/web/compressed.tracemonkey-pldi-09.pdf';
		elasticpdf_viewer.pdfAnnotation.refreshFabricState(1);
		elasticpdf_viewer.pdfAnnotation.openFile(test_pdf);
		console.log('pdfurl', elasticpdf_viewer.PDFViewerApplication.baseUrl);
	}

	//设置用户
	function setUser(id) {
		if (is_elasticpdf.value==false){
			alert('only available for elasticpdf');
			return;
		}
		var this_member = {
			'id': id,
			'name': '张三的名字',
		};
		elasticpdf_viewer.setCurrentMemberId(this_member);
	}


	async function reloadData() {
		if (is_elasticpdf.value==false){
			alert('only available for elasticpdf');
			return;
		}
		let file_name = 'tutorial.pdf'
		let annotation_content = await postService('get-annotation-data', {
			'file_name': 'tutorial.pdf',
			'file_id': '123ddasfsdffads',
		});
		// 批注重载回显于当前文件
		if (annotation_content){
			elasticpdf_viewer.setPureFileAnnotation({
				'file_annotation': annotation_content
			});
		}
		
	}


	// 与后端服务器进行网络通信的函数
	async function postService(url, data) {
		return null;
		var new_data = new URLSearchParams();
		var encrpte_data = data;
		new_data.append('data', encrpte_data);

		var base_url = "your-server-url";
		var posturl = base_url + url;
		const response = await fetch(posturl, {
			method: 'POST',
			headers: {},
			body: new_data,
		});


		const resp = await response.json();
		resp['data'] = JSON.parse(resp['data']);

		return resp;
	}
</script>

<style>
	.project-title {
		display: flex;
		justify-content: space-between;
		flex-direction: row;
		align-items: center;
		background-color: aliceblue;
		border-radius: 5px;
		height: 50px;
		padding: 0 20px 0px 10px;
		position: relative;
	}

	.project-title div {
		height: 50px;
		padding: 0 20px 0px 10px;
		align-items: center;
		display: flex;
		flex-direction: row;
	}

	.project-title h2 {
		width: auto;
		text-align: left;
		color: var(--blue-500);
		margin-left: 20px;
		margin-right: 20px;
	}

	.project-title div button {
		margin-left: 10px;
	}

	.project-title img {
		margin-left: 10px;
		height: 80%;
	}

	.project-title div img {
		margin-left: 20px;
		height: 80%;
	}



	:root {
		--blue-50: #f5f9ff;
		--blue-100: #d0e1fd;
		--blue-200: #abc9fb;
		--blue-300: #85b2f9;
		--blue-400: #609af8;
		--blue-500: #3b82f6;
		--blue-600: #326fd1;
		--blue-700: #295bac;
		--blue-800: #204887;
		--blue-900: #183462;
		--green-50: #f4fcf7;
		--green-100: #caf1d8;
		--green-200: #a0e6ba;
		--green-300: #76db9b;
		--green-400: #4cd07d;
		--green-500: #22c55e;
		--green-600: #1da750;
		--green-700: #188a42;
		--green-800: #136c34;
		--green-900: #0e4f26;
		--yellow-50: #fefbf3;
		--yellow-100: #faedc4;
		--yellow-200: #f6de95;
		--yellow-300: #f2d066;
		--yellow-400: #eec137;
		--yellow-500: #eab308;
		--yellow-600: #c79807;
		--yellow-700: #a47d06;
		--yellow-800: #816204;
		--yellow-900: #5e4803;
		--cyan-50: #f3fbfd;
		--cyan-100: #c3edf5;
		--cyan-200: #94e0ed;
		--cyan-300: #65d2e4;
		--cyan-400: #35c4dc;
		--cyan-500: #06b6d4;
		--cyan-600: #059bb4;
		--cyan-700: #047f94;
		--cyan-800: #036475;
		--cyan-900: #024955;
		--pink-50: #fef6fa;
		--pink-100: #fad3e7;
		--pink-200: #f7b0d3;
		--pink-300: #f38ec0;
		--pink-400: #f06bac;
		--pink-500: #ec4899;
		--pink-600: #c93d82;
		--pink-700: #a5326b;
		--pink-800: #822854;
		--pink-900: #5e1d3d;
		--indigo-50: #f7f7fe;
		--indigo-100: #dadafc;
		--indigo-200: #bcbdf9;
		--indigo-300: #9ea0f6;
		--indigo-400: #8183f4;
		--indigo-500: #6366f1;
		--indigo-600: #5457cd;
		--indigo-700: #4547a9;
		--indigo-800: #363885;
		--indigo-900: #282960;
		--teal-50: #f3fbfb;
		--teal-100: #c7eeea;
		--teal-200: #9ae0d9;
		--teal-300: #6dd3c8;
		--teal-400: #41c5b7;
		--teal-500: #14b8a6;
		--teal-600: #119c8d;
		--teal-700: #0e8174;
		--teal-800: #0b655b;
		--teal-900: #084a42;
		--orange-50: #fff8f3;
		--orange-100: #feddc7;
		--orange-200: #fcc39b;
		--orange-300: #fba86f;
		--orange-400: #fa8e42;
		--orange-500: #f97316;
		--orange-600: #d46213;
		--orange-700: #ae510f;
		--orange-800: #893f0c;
		--orange-900: #642e09;
		--bluegray-50: #f7f8f9;
		--bluegray-100: #dadee3;
		--bluegray-200: #bcc3cd;
		--bluegray-300: #9fa9b7;
		--bluegray-400: #818ea1;
		--bluegray-500: #64748b;
		--bluegray-600: #556376;
		--bluegray-700: #465161;
		--bluegray-800: #37404c;
		--bluegray-900: #282e38;
		--purple-50: #fbf7ff;
		--purple-100: #ead6fd;
		--purple-200: #dab6fc;
		--purple-300: #c996fa;
		--purple-400: #b975f9;
		--purple-500: #a855f7;
		--purple-600: #8f48d2;
		--purple-700: #763cad;
		--purple-800: #5c2f88;
		--purple-900: #432263;
		--red-50: #fff5f5;
		--red-100: #ffd0ce;
		--red-200: #ffaca7;
		--red-300: #ff8780;
		--red-400: #ff6259;
		--red-500: #ff3d32;
		--red-600: #d9342b;
		--red-700: #b32b23;
		--red-800: #8c221c;
		--red-900: #661814;
		--primary-50: #f3fcf9;
		--primary-100: #c6eee1;
		--primary-200: #98e1c9;
		--primary-300: #6bd4b1;
		--primary-400: #3dc699;
		--primary-500: #10b981;
		--primary-600: #0e9d6e;
		--primary-700: #0b825a;
		--primary-800: #096647;
		--primary-900: #064a34;
	}


	.p-overlaypanel-content {
		border-radius: 10px;
		position: absolute;
		background-color: #fff;
		padding: 10px;
		box-shadow: 0px 0px 10px rgb(0 0 0 / 15%);
		z-index: 100000;
	}


	.triangle-down {
		display: block;
		width: 0;
		height: 0;
		border-width: 10px 10px 0;
		position: absolute;
		left: 20px;
		top: -10px;
		transform: scaleY(-1);
		border-style: solid;
		/* 红 透明 透明 */
		border-color: #dedede transparent transparent transparent;
	}

	.triangle-down em {
		display: block;
		width: 0;
		height: 0;
		border-width: 10px 10px 0;
		position: absolute;
		left: -10px;
		top: -11px;
		border-style: solid;
		/* 红 透明 透明 */
		border-color: #fff transparent transparent transparent;
	}

	.triangle-right {
		display: block;
		width: 0;
		height: 0;
		border-width: 10px 10px 0;
		position: absolute;
		left: -15px;
		top: 15px;
		transform: scaleY(-1) rotate(270deg);
		border-style: solid;
		/* 红 透明 透明 */
		border-color: #dedede transparent transparent transparent;
	}

	.triangle-right em {
		display: block;
		width: 0;
		height: 0;
		border-width: 10px 10px 0;
		position: absolute;
		left: -10px;
		top: -10px;
		border-style: solid;
		/* 红 透明 透明 */
		border-color: #fff transparent transparent transparent;
	}

	.theme-btn {
		border-radius: 12px;
		text-align: center;
		border: none;
		max-width: max-content;
		width: auto;
		padding: 5px 15px;
		position: relative;
		display: flex;
		align-items: center;
		justify-content: center;
		font-family: 'Inter-Regular';
		font-weight: normal;
		display: flex;
		justify-content: space-between;
	}

	.btn-main {
		color: #fff;
		background: var(--theme-color-1);
		transition: all .5s ease-out;
		white-space: nowrap;
		min-width: 100px;
	}

	.btn-main:hover {
		background-position: left bottom;
		color: #fff;
		background: #1a7967;
		transition: all .5s ease-out;
	}

	.btn-secondary {
		color: var(--font-white);
		background-color: var(--theme-color-2);
		transition: all .5s ease-out;
	}

	.btn-secondary:hover {
		background-position: left bottom;
		color: #fff;
		background: #171389;
		transition: all .5s ease-out;
	}

	.btn-success {
		font-weight: 700;
		color: var(--font-white);
		background-color: var(--green-500);
		transition: all .5s ease-out;
	}

	.btn-success:hover {
		background-position: left bottom;
		color: #fff;
		background: var(--green-400);
		transition: all .5s ease-out;
	}

	.btn-outline-success {
		font-weight: 700;
		color: var(--green-500);
		border: 1.5px solid var(--green-500);
		transition: all .5s ease-out;
		padding: 5px 15px 5px 15px;
		border-radius: 25px;
		background-color: rgba(255, 255, 255, 0);
	}

	.btn-outline-success:hover {
		font-weight: 700;
		color: var(--green-400);
		border: 1.5px solid var(--green-400);
		transition: all .5s ease-out;
	}

	.btn-info {
		font-weight: 700;
		color: var(--font-white);
		background-color: var(--blue-500);
		transition: all .5s ease-out;
	}

	.btn-info:hover {
		background-position: left bottom;
		color: #fff;
		background: var(--blue-400);
		transition: all .5s ease-out;
	}

	.btn-outline-info {
		font-weight: 700;
		color: var(--blue-500);
		border: 1.5px solid var(--blue-500);
		transition: all .5s ease-out;
		padding: 5px 15px 5px 15px;
		border-radius: 25px;
		background-color: rgba(255, 255, 255, 0);
	}

	.btn-outline-info:hover {
		font-weight: 700;
		color: var(--blue-400);
		border: 1.5px solid var(--blue-400);
		transition: all .5s ease-out;
	}

	.btn-danger {
		font-weight: 700;
		color: var(--font-white);
		background-color: var(--red-500);
		transition: all .5s ease-out;
	}

	.btn-danger:hover {
		background-position: left bottom;
		color: #fff;
		background: var(--red-400);
		transition: all .5s ease-out;
	}

	.btn-outline-danger {
		font-weight: 700;
		color: var(--red-500);
		border: 1.5px solid var(--red-500);
		transition: all .5s ease-out;
		padding: 5px 15px 5px 15px;
		border-radius: 25px;
		background-color: rgba(255, 255, 255, 0);
	}

	.btn-outline-danger:hover {
		font-weight: 700;
		color: var(--red-400);
		border: 1.5px solid var(--red-400);
		transition: all .5s ease-out;
	}

	.btn-help {
		font-weight: 700;
		color: var(--font-white);
		background-color: var(--purple-500);
		transition: all .5s ease-out;
	}

	.btn-help:hover {
		background-position: left bottom;
		color: #fff;
		background: var(--purple-400);
		transition: all .5s ease-out;
	}

	.btn-outline-help {
		font-weight: 700;
		color: var(--purple-500);
		border: 1.5px solid var(--purple-500);
		transition: all .5s ease-out;
		padding: 5px 15px 5px 15px;
		border-radius: 25px;
		background-color: rgba(255, 255, 255, 0);
	}

	.btn-outline-help:hover {
		font-weight: 700;
		color: var(--purple-400);
		border: 1.5px solid var(--purple-400);
		transition: all .5s ease-out;
	}

	.btn-warning {
		color: var(--font-black);
		background-color: var(--orange-500);
		transition: all .5s ease-out;
	}

	.btn-warning:hover {
		background-position: left bottom;
		color: #fff;
		background: var(--orange-400);
		transition: all .5s ease-out;
	}

	.btn-outline-warning {
		font-weight: 700;
		color: var(--orange-500);
		border: 1.5px solid var(--orange-500);
		transition: all .5s ease-out;
		padding: 5px 15px 5px 15px;
		border-radius: 25px;
		background-color: rgba(255, 255, 255, 0);
	}

	.btn-outline-warning:hover {
		font-weight: 700;
		color: var(--orange-400);
		border: 1.5px solid var(--orange-400);
		transition: all .5s ease-out;
	}

	.btn-no-design {
		color: #000000;
		transition: ease-in 0.3s;
	}

	.btn-no-design i {
		margin-left: 8px;
	}

	.btn-no-design:hover {
		color: var(--theme-color-2);
		transition: ease-in 0.3s;
	}
</style>