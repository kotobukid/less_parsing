<script setup lang="ts">
import {type Ref, ref} from "vue";
import less from "less";

const source_text = ref(`
a {color: red;
    span {color: green;}
}
.circle-a {fill: red; stroke: black; stroke-width: 2px;}
.circle-b {fill: blue; stroke: white; stroke-width: 1px;}
rect.back {fill: pink;}
`);
const parsed_text = ref("");

const originalSVG: Ref<SVGSVGElement | null> = ref(null);
const example1: Ref<HTMLImageElement | null> = ref(null);
const example2: Ref<HTMLImageElement | null> = ref(null);

type CssRuleTree = Record<string, Record<string, string>>;

const parse_css = (css: string): CssRuleTree => {
    const rules: CssRuleTree = {};
    const blocks: string[] = css.split('}');
    for (let i: number = 0; i < blocks.length; i++) {

        let rule: Record<string, string> = {};
        let lines: string[] = blocks[i].split('\n');
        let selector: string
        for (let j = 0; j < lines.length; j++) {
            selector = lines.shift().replace(/^\n/, '').replace(/\{/, '').trim();
            if (selector) {
                break;
            }
        }

        if (selector) {
            for (let j = 0; j < lines.length; j++) {
                let [prop_name, value] = lines[j].split(':');
                prop_name = (prop_name || '').trim();
                value = (value || '').trim();
                if (prop_name && value) {
                    rule[prop_name] = value.replace(/;$/, '');
                }
            }
            rules[selector] = rule;
        }
    }
    return rules;
};

const test = () => {
    const css: string = `a {
      color: red;
    }
    a span {
      color: green;
    }
    .circle-a {
      fill: red;
      stroke: black;
      stroke-width: 2px;
    }
    .circle-b {
      fill: blue;
      stroke: white;
      stroke-width: 1px;
    }
    rect.back {
      fill: pink;
    }`;


    console.log(parse_css(css));
};

// test();

const apply = () => {
    less.render(source_text.value).then(({css}: { css: string }) => {
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

        const cssRules: CssRuleTree = parse_css(css);

        if (originalSVG.value) {
            const svg_source = originalSVG.value.outerHTML;
            const parser = new DOMParser();
            const doc = parser.parseFromString(svg_source, 'image/svg+xml');

            // @ts-ignore
            const cloned_svg_element: SVGSVGElement = <SVGSVGElement>doc.documentElement;
            cloned_svg_element.setAttribute("xmlns", "http://www.w3.org/2000/svg");

            if (example1.value) {
                render_as_bmp(example1, cloned_svg_element);
            }

            cloned_svg_element.querySelectorAll('*').forEach((element) => {
                for (const selector in cssRules) {
                    if (element.matches(selector)) {
                        const properties = cssRules[selector];
                        for (const [property, value] of Object.entries(properties)) {
                            // if (!element.style) {
                            //     element.style = {};
                            // }
                            element.setAttribute(property, <string>value);
                        }
                    }
                }
            });

            if (example2.value) {
                render_as_bmp(example2, cloned_svg_element);
            }

        }
    }, (error: Error) => {
        console.error(error);
    });
}

const render_as_bmp = (elem_ref: Ref<HTMLImageElement | null>, svg_element: SVGSVGElement) => {
    const svg_str = '<?xml version="1.0" encoding="utf-8"?>' + new XMLSerializer().serializeToString(svg_element);

    const img = new Image();
    const canvas = document.createElement('canvas');
    const ctx: CanvasRenderingContext2D = canvas.getContext('2d')!;

    img.onload = () => {
        canvas.width = img.width;
        canvas.height = img.height;
        ctx.drawImage(img, 0, 0);
        if (elem_ref.value) {

            elem_ref.value.src = canvas.toDataURL('image/png');
        }
    };

    img.onerror = (e) => {
        console.error(e);
    }

    img.src = 'data:image/svg+xml;base64,' + btoa(svg_str);

}

</script>

<template lang="pug">
.less_parser
    .wrapper
        textarea.main_input(v-model="source_text")
        a.button.apply_less(href="#" @click.prevent="apply")
            span Lessをパースし
            br
            span ページに適用
        textarea.main_output(v-model="parsed_text" readonly="readonly")
        br.clearfix
    br
    table
        tr
            th
                span オリジナルSVG要素
                br
                span (style要素使用)
            td
                svg.render.svg_sample(ref="originalSVG" width="300" height="120" viewBox="0 0 300 120")
                    rect.back(x="0" y="0" width="300" height="120" fill="white")
                    circle.circle-a(r="40" cx="50" cy="50")
                    circle.circle-b(r="40" cx="250" cy="50")
        tr
            th
                span Canvas経由でDataURL転写
            td
                img.render.example1(ref="example1")
        tr
            th
                span スクリプトでスタイルルールをインライン化して転写
            td
                img.render.example2(ref="example2")

</template>

<style scoped lang="less">
.clearfix {
    clear: both;
}

.wrapper {
    width: 1000px;

    & > * {
        float: left;
    }

    a.apply_less {
        display: inline-block;
        width: 200px;
        text-align: center;
        padding: 5px;
        border-radius: 5px;
        border: 1px solid green;
        background-color: white;

        &:hover {
            background-color: lightblue;
        }

        &:active {
            position: relative;
            top: 1px;
            background-color: lightgreen;
        }
    }

    a.button {
        margin-right: 5px;
    }

    margin-bottom: 10px;
}

.main_input, .main_output {
    width: 300px;
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

.render {
    outline: 1px solid grey;
    width: 300px;
    height: 120px;
    margin: 5px;
}

rect.back {
    fill: lightblue;
}

table {
    table-layout: fixed;
    border-collapse: collapse;
}

th, td {
    border: 1px solid grey;
}
</style>