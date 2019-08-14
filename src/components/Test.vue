<template>
    <div class="hello">
        <div class="item" v-for="(node, index) in nodes" :key="index">
            <div class="desc">{{ node.desc }}</div>
            <div class="title">{{ node.title }}</div>
        </div>
        <svg id="container"></svg>
    </div>
</template>

<script>
import { Core, Node, Edge } from "flowchart-core";
import nodes from "../config/nodes.json";

const width = 100;
const height = 100;

export default {
    data() {
        return {
            container: {},
            nodes: nodes
        };
    },
    mounted() {
        const svgContainer = document.getElementById("container");

        this.container = new Core(svgContainer, {
            style: {
                width: "100vw",
                height: "100vh",
                border: "10px solid #000"
            },
            line: {
                style: {
                    stroke: "deepskyblue"
                },
                arrow: {
                    style: {
                        fill: "#888"
                    },
                    viewBox: "0 0 18 18"
                }
            },
            linkDot: {
                display: "none"
            },
            control: true // must need. default is false 是否可配置流程
        });

        // 初始化节点布局
        this.intiialLayout(this.nodes);
    },
    methods: {
        findChilds(nodes, node) {
            // 此 node 具备两个条件：1. 它是当前父节点下的子节点 2. 它有子节点
            // 目标：将它下列的所有子孙节点找出
            let cs = [];
            const childs = nodes.filter(n => {
                if (n.source === node.id) {
                    if (n.children.length > 0) {
                        cs = this.findChilds(nodes, n);
                    }
                    return n;
                }
            });
            return Array.from(new Set([...childs, ...cs]));
        },
        intiialLayout(nodes) {
            // 找出根节点
            const rootNode = nodes.find(node => {
                if (node.children.length > 0) {
                    const index = nodes.findIndex(
                        n => node.children.indexOf(n.id) !== -1
                    );
                    // debugger
                    if (index !== -1) {
                        return node;
                    }
                    // TODO: find root!
                    // let result = nodes.reduce((acc, curv) => {
                    //     if (
                    //         acc.children.length > 0 &&
                    //         node.children.indexOf(acc.id) !== -1
                    //     ) {
                    //         return acc;
                    //     } else {
                    //         return curv;
                    //     }
                    // });
                    // debugger
                    // return result;
                }
            });
            // debugger;
            const rootNodeExistIndex = this.container.nodes.findIndex(
                node => node.id === rootNode.id
            );
            if (rootNodeExistIndex === -1) {
                this.createNode(rootNode);
            }
            // 将后续子孙节点注入队列中
            let nodeQueue = [];
            nodes.forEach(node => {
                if (node.source === rootNode.id) {
                    if (node.children.length > 0) {
                        let list = nodes.filter(
                            n => node.children.indexOf(n.id) !== -1
                        );
                        const childs = this.findChilds(nodes, node);
                        list = Array.from(new Set([node, ...list, ...childs]));
                        nodeQueue.push(list);
                        // debugger;
                    }
                }
            });
            // debugger;
            // 筛选子节点
            const childNodes = nodes.filter(
                node => node.source === rootNode.id
            );
            // 创建子节点
            childNodes.forEach((node, index) => {
                const len = node.children.length;
                // 根据子节点个数生成父节点位置
                if (len > 0) {
                    node.position = this.initialPosition(
                        node,
                        rootNode,
                        index + (len % index)
                    );
                } else {
                    node.position = this.initialPosition(node, rootNode, index);
                }
                this.createNode(node);
            });
            // 绘制连接
            this.drawLink(childNodes, rootNode);
            // 递归子节点包含有子孙节点情况
            if (nodeQueue.length > 0) {
                nodeQueue.forEach(list => {
                    this.intiialLayout(list);
                });
            }
        },
        /**
         * 初始化position属性
         */
        initialPosition(node, rootNode, index) {
            const baseWidth = width + width / 2;
            const baseHeight = height + height / 2;
            const { x: baseX, y: baseY } = rootNode.position;
            node.position.x = baseX + baseWidth * index;
            node.position.y = baseY + baseHeight;
            return node.position;
        },
        /**
         * 绘制连接关系
         */
        drawLink(nodes, rootNode) {
            const self = this;
            nodes.forEach(node => {
                let edge = new Edge();
                Object.assign(edge, {
                    source: rootNode.id,
                    target: node.id,
                    dotLink: "bottom",
                    dotEndLink: "top"
                });
                edge.lineData = self.container.edgeData(edge);

                self.container.edges = [edge, ...self.container.edges];
                self.container.edgeG.appendChild(edge.edge);
            });
        },
        /**
         * 创建节点
         */
        createNode(node) {
            const htmlData = document.querySelector(".item");
            let nodeInstance = new Node({
                position: node.position,
                style: {
                    width,
                    height,
                    userSelect: "none",
                    rx: 10
                },
                html: {
                    meta: htmlData
                }
            });
            Object.assign(nodeInstance, node);
            this.container.addNode(nodeInstance);
        }
    }
};
</script>

<style scoped>
div.item {
    width: 100px;
    height: 100px;
    color: black;
    font-size: 24px;
    font-weight: bold;
    background-color: #9c3;
    border-radius: 4px;
    display: flex;
    flex-flow: column;
    align-items: center;
    justify-content: center;
}
</style>
