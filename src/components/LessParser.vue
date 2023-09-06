<script setup lang="ts">
import {ref} from "vue";
import less from "less";

const source_text = ref(`
a {color: red;
    span {color: green;}
}
.circle-a {fill: red; stroke: black; stroke-width: 2px;}
.circle-b {fill: blue; stroke: white; stroke-width: 1px;}
`);
const parsed_text = ref("");

const parse = () => {
    console.log(source_text.value);
    less.render(source_text.value).then(({css, imports}: {css: string, imports: any[]}) => {
        console.log(imports);
        parsed_text.value = css;

        const my_style_element = document.getElementById('my-style-element');
        if (my_style_element) {
            my_style_element.textContent = css;
        } else {
            const mse = document.createElement("style");
            mse.setAttribute('id', 'my-style-element');
            mse.textContent = css;
            document.head.appendChild(mse);
        }

    }, (error: Error) => {
        console.error(error);
    });
}

</script>

<template lang="pug">
.less_parser
    .wrapper
        textarea.main_input(v-model="source_text")
        a.button(href="#" @click.prevent="parse") parse
        textarea.main_output(v-model="parsed_text" readonly="readonly")
        br.clearfix
    br
    svg.svg_sample(width="400" height="200" viewBox="0 0 400 200")
        circle.circle-a(r="40"  cx="50" cy="50")
        circle.circle-b(r="40"  cx="250" cy="50")

</template>

<style scoped lang="less">
.clearfix {
    clear: both;
}
.wrapper {
    width: 1000px;
    &>* {
        float: left;
    }
    a {
        display: inline-block;
        width: 60px;
        text-align: center;
    }
    margin-bottom: 10px;
}
.main_input, .main_output {
    width: 400px;
    height: 200px;
    border: 1px solid black;
    padding: 10px;
    border-radius: 10px;
}
.main_input {
    margin-right: 10px;
}

.main_output {
    background-color: #efefef;
}

.svg_sample {
    outline: 1px solid grey;
}
</style>