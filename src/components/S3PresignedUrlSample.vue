<template>
	<div class="hello">
		<input id="upload" type="file" ref="file" @input="upload" multiple />
		<input id="delete" type="button" @click="executeDeleteFile" value="削除" />
		<input
			id="delete"
			type="button"
			@click="executeDownloadFile"
			value="ダウンロード"
		/>
	</div>
</template>

<script>
import axios from "axios";
import { S3RequestPresigner } from "@aws-sdk/s3-request-presigner";
import { HttpRequest } from "@aws-sdk/protocol-http";
import { parseUrl } from "@aws-sdk/url-parser";
import { formatUrl } from "@aws-sdk/util-format-url";
import { Sha256 } from "@aws-crypto/sha256-js";
export default {
	name: "S3PresignedUrlSample",
	props: {
		msg: String,
	},
	data() {
		return {
			count: 0,
		};
	},
	methods: {
		async executeDownloadFile() {
			try {
				const signedUrl = await this.createPresignedUrl(
					undefined,
					undefined,
					undefined,
					"GET"
				);
				location.href = signedUrl;
			} catch (e) {
				console.error(e);
			}
		},
		async executeUploadFile() {
			try {
				const tasks = new Array(...this.$refs.file.files).map(
					async (element) => {
						const objectKey = "folder/" + element.name;
						const signedUrl = await this.createPresignedUrl(
							objectKey,
							undefined,
							undefined,
							"PUT"
						);
						// アップロード時のHTTPリクエストヘッダは必ず署名付きURLを要求した時のものにする。
						// 署名付きURLを取得した時と異なるヘッダ情報を付与して要求した場合、403:fobiddenが発生する。
						return axios.put(signedUrl, element);
					}
				);
				Promise.all([...tasks]).then((response) => {
					console.log(response);
				});
			} catch (e) {
				console.error(e);
			}
		},
		async executeDeleteFile() {
			try {
				const signedUrl = await this.createPresignedUrl(
					undefined,
					undefined,
					undefined,
					"DELETE"
				);

				let response = await axios.delete(signedUrl);
				console.log(response);
			} catch (e) {
				console.error(e);
			}
		},
		upload() {
			this.executeUploadFile(this.$refs.file.files[0]);
		},
		async createPresignedUrl(
			objectKey = "objectKey",
			region = "ap-northeast-1",
			bucket = "bucket",
			httpMethod
		) {
			const url = parseUrl(
				`https://${bucket}.s3.${region}.amazonaws.com/${objectKey}`
			);
			const presigner = new S3RequestPresigner({
				credentials: {
					accessKeyId: "accessKeyId",
					secretAccessKey: "secretAccessKey",
				},
				region,
				sha256: Sha256,
			});
			const signedUrlObject = await presigner.presign(
				new HttpRequest({ ...url, method: httpMethod })
			);
			return formatUrl(signedUrlObject);
		},
	},
};
</script>
<style scoped></style>
