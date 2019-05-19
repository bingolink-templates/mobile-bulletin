<template>
    <div style="flex:1">
        <notice></notice>
    </div>
</template>

<!--引入bui-weex样式文件-->
<style lang="scss" src="bui-weex/src/css/buiweex.scss"></style>

<script>
    import notice from "./notice.vue";
    module.exports = {
        data () {
            return {}
        },
        components: {
          notice
        }
    }
</script>