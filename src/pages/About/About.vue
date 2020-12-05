<template>
    <div class="about">
        <div class="about-item"><img src="../../assets/logo.png" width="100" height="100" alt="logo"></div>
        <div class="about-item">{{ name }} - {{ version }}</div>
        <div class="about-item">Author: {{ author }}</div>
        <div class="about-item">
            <el-button size="mini" type="success" @click="openUrl(blog)">博客</el-button>
            <el-button size="mini" type="danger" @click="openUrl(homepage)">项目主页</el-button>
        </div>
    </div>
</template>

<script>
    import config from '../../../package'
    export default {
        name: "About",
        data() {
            return {
                name: '',
                version: '',
                author: '',
                homepage: '',
                blog: '',
            }
        },
        created() {
            this.name = config.name;
            this.version = config.version;
            this.author = config.author;
            this.homepage = config.homepage;
            this.blog = config.blog;
        },
        methods: {
            openUrl(url) {
                electron.shell.openExternal(url);
            },
        }
    }
</script>

<style lang="scss" scoped>
    .about {
        text-align: center;
        .about-item {
            padding: 10px;
        }
    }
</style>
