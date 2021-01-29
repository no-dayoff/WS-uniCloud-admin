<template>
    <scroll-view style="padding: 15px;box-sizing: border-box;">
        内容主体，可自定义内容及样式<text style="color: #666; font-size: 15px;">（ 页面初始文字）</text>
		<!-- #ifndef H5 -->
		<fix-window />
		<!-- #endif -->
    </scroll-view>
</template>

<script>
    export default {
        data() {
            return {
			}
        },
        onLoad() {},
        methods: {}
    }
</script>

<style>
	/* #ifndef H5 */
	page {
		padding-top: 85px;
	}
	/* #endif */
</style>
