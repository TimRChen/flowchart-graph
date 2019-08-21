<template>
    <div class="hello">
        <div
            class="item"
            v-for="(node, index) in nodes"
            :data-rsgraph-id="node.id"
            :key="index"
        >
            <div class="desc">{{ node.desc }}</div>
            <div class="title">{{ node.title }}</div>
        </div>
        <div class="node-zone">
            <svg
                id="container"
                width="100vw"
                height="100vh"
                viewBox="0 0 1000 1000"
                preserveAspectRatio="xMinYMin meet"
            ></svg>
        </div>
    </div>
</template>

<script>
import { RSGraph } from "flowchart-core";
import nodes from "../config/nodes.json";

export default {
    data() {
        return {
            container: {},
            nodes: nodes.sort(() => (Math.random() * 10 > 5 ? -1 : 1)) // random list
            // nodes: nodes.sort((a, b) => (a.title > b.title ? 1 : -1)) // 排序
        };
    },
    mounted() {

        const width = 50;
        const height = 50;
        const nodes = this.nodes.map(node => {
            node.width = width;
            node.height = height;
            return node;
        });

        const graph = new RSGraph("#container", {
            data: nodes
        });
    }
};
</script>

<style scoped>
div.item {
    width: 50px;
    height: 50px;
    color: #fff;
    font-size: 12px;
    font-weight: bold;
    background-color: #000;
    border-radius: 4px;
    display: flex;
    flex-flow: column;
    align-items: center;
    justify-content: center;
}

div.node-zone {
    width: 100%;
    overflow: scroll;
}
</style>
