<template>
    <div class="hello">
        <div
            class="item"
            v-for="(node, index) in nodes"
            :data-id="node.id"
            :key="index"
        >
            <div class="desc">{{ node.desc }}</div>
            <div class="title">{{ node.title }}</div>
        </div>
        <div class="node-zone">
            <svg
                id="container"
                width="2000"
                height="1000"
                viewBox="0 0 1000 1000"
                preserveAspectRatio="xMinYMin meet"
            ></svg>
        </div>
    </div>
</template>

<script>
import { Core, Node, Edge } from "flowchart-core";
import nodes from "../config/nodes.json";

const width = 50;
const height = 50;

export default {
    data() {
        return {
            container: {},
            // nodes,
            nodes: nodes.sort(() => (Math.random() * 10 > 5 ? -1 : 1)) // random list
        };
    },
    mounted() {
        const svgContainer = document.getElementById("container");

        this.container = new Core(svgContainer, {
            style: {
                border: "1px solid #000",
                overflow: "scroll"
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
            mode: "link-mode" // must need. default is render-mode 是否可配置流程
        });

        // 初始化节点布局
        this.intiialLayout(this.nodes);
    },
    methods: {
        findRoot(nodes) {
            const rootNode = nodes.find(node => {
                if (node.children.length > 0) {
                    const index = nodes.findIndex(n => node.parent === n.id);
                    return index === -1;
                }
            });
            return rootNode;
        },
        findChilds(nodes, node) {
            // 此 node 具备两个条件：1. 它是当前父节点下的子节点 2. 它有子节点
            // 目标：将它下列的所有子孙节点找出
            let cs = [];
            const childs = nodes.filter(n => {
                if (n.parent === node.id) {
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
            const rootNode = this.findRoot(nodes);
            // 判断根节点是否已存在
            const rootIndex = this.container.nodes.findIndex(
                node => node.id === rootNode.id
            );
            if (rootIndex === -1) {
                // 若根节点不存在则创建
                this.createNode(rootNode);
            }
            // 筛选子节点
            const childNodes = nodes.filter(
                node => node.parent === rootNode.id
            );
            // 将后续子孙节点注入递归队列中
            let nodeQueue = [];
            childNodes.forEach(node => {
                if (node.parent === rootNode.id) {
                    if (node.children.length > 0) {
                        let list = childNodes.filter(
                            n => node.children.indexOf(n.id) !== -1
                        );
                        const childs = this.findChilds(this.nodes, node);
                        list = Array.from(new Set([node, ...list, ...childs]));
                        nodeQueue.push(list);
                    }
                }
            });

            // 创建子节点并初始化位置 TODO: position is the key.
            let lastNodeLen = 0;
            childNodes.forEach((node, index) => {
                const nodeInstance = this.createNode(node);
                const len = node.children.length;
                // 根据子节点个数生成父节点位置
                if (len >= 1 && lastNodeLen > 0) {
                    nodeInstance.changePosition(
                        this.initialPosition(node, rootNode, lastNodeLen)
                    );
                } else {
                    nodeInstance.changePosition(
                        this.initialPosition(node, rootNode, index)
                    );
                }
                lastNodeLen += len + index;
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
        initialPosition(node, rootNode, len) {
            const baseWidth = width * 2;
            const baseHeight = height * 2;
            const { x: baseX, y: baseY } = rootNode.position;
            const xPosition = baseX + baseWidth * len;
            const yPosition = baseY + baseHeight;
            node.position.x = xPosition;
            node.position.y = yPosition;
            return node.position;
        },
        /**
         * 绘制连接关系
         */
        drawLink(nodes, rootNode) {
            nodes.forEach(node => {
                const edge = new Edge();
                this.container.addEdge(edge, {
                    source: rootNode.id,
                    target: node.id,
                    dotLink: "bottom",
                    dotEndLink: "top"
                });
            });
        },
        /**
         * 创建节点
         */
        createNode(node) {
            const domSelectors = `.item[data-id="${node.id}"]`;
            const htmlData = document.querySelector(domSelectors);
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
            return nodeInstance;
        }
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
