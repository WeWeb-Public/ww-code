<template>
    <div class="ww-markdown">
        <div class="filler-object ww-markdown-edit-mode">
            <pre v-highlightjs="markdownText"><code spellcheck="false" class="ww-markdown-box" :class="languageStyle"></code></pre>
        </div>
    </div>
</template>
 

<script>			
import Vue from 'vue';
import VueHighlightJS from 'vue-highlightjs'
import hljs from 'highlight.js'
//import 'highlight.js/styles/github.css'
import 'highlight.js/styles/atom-one-dark.css'
//https://highlightjs.org/static/demo/

Vue.use(VueHighlightJS)

export default {
    name: "__COMPONENT_NAME__",
    props: {
        wwObjectCtrl: Object,
        wwAttrs: {
            type: Object,
            default: {}
        }
    },
    data() {
        return {
            languageList: [],
        }
    },
    computed: {
        wwObject() {
            return this.wwObjectCtrl.get();
        },
        editMode() {
            return this.wwObjectCtrl.getSectionCtrl().getEditMode() == 'CONTENT'
        },
        markdownText() {
            if (this.wwObject) {
                return this.wwObject.content.data.markdownText
            } else {
                return ""
            }
        },
        languageStyle() {
            if (this.wwObject) {
                return this.wwObject.content.data.languageStyle
            } else {
                return ""
            }
        }

        /*  languageList() {
             return (hljs.listLanguages()).sort() //TODO: one day use this to get value from list 
         } */
    },
    watch: {
    },

    methods: {
        init() {
            this.languageList = (hljs.listLanguages()).sort()

            this.loaded = true
            this.wwObject.content.data = this.wwObject.content.data || {}

            if (!this.wwObject.content.data.markdownText) {
                this.wwObject.content.data.markdownText = wwLib.wwObject.getDefault({
                    type: 'ww-text',
                });
            }
            if (!this.wwObject.content.data.languageStyle) {
                this.wwObject.content.data.languageStyle = wwLib.wwObject.getDefault({
                    type: 'ww-text',
                });
            }
            this.wwObjectCtrl.update(this.wwObject)
        },
        async edit() {
            wwLib.wwObjectHover.setLock(this);
            let editList = {
                WWMARK_CONFIG: {
                    title: {
                        en: 'Edit code',
                        fr: 'Editer le code'
                    },
                    desc: {
                        en: 'Edit code to be formatted',
                        fr: 'Editer le code à formatter'
                    },
                    icon: 'wwi wwi-laptop',
                    shortcut: 'r',
                    next: 'WWMARK_CODE'
                },

            }

            wwLib.wwPopups.addStory('WWMARK_CONFIG', {
                title: {
                    en: 'Configuration',
                    fr: 'Configurer'
                },
                type: 'wwPopupEditWwObject',
                buttons: null,
                storyData: {
                    list: editList
                }
            })

            wwLib.wwPopups.addStory('WWMARK_CODE', {
                title: {
                    en: 'Fill in code',
                    fr: 'Inserer le code'
                },
                type: 'wwPopupForm',
                storyData: {
                    fields: [
                        {
                            label: {
                                en: 'Programming Language:',
                                fr: 'Langage de programmation :'
                            },
                            type: 'text',
                            key: 'languageStyle',
                            valueData: 'languageStyle',
                            desc: {
                                en: 'Exp: css, dockerfile, http, html, javascript, json, php, sql, scss, typescript',
                                fr: 'Ex: css, dockerfile, http, html, javascript, json, php, sql, scss, typescript'
                            }
                            //https://highlightjs.readthedocs.io/en/latest/css-classes-reference.html list of all PL that are handeled by highlight
                        },
                        {
                            label: {
                                en: 'Code:',
                                fr: 'Code :'
                            },
                            type: 'textarea',
                            key: 'markdownText',
                            valueData: 'markdownText',
                            desc: {
                                en: '',
                                fr: ''
                            },
                            style: {
                                height: '600px'
                            }
                        },

                    ]
                },
                buttons: {
                    NEXT: {
                        text: {
                            en: 'Finish',
                            fr: 'Terminer'
                        },
                        next: null
                    }
                }
            })

            let options = {
                firstPage: 'WWMARK_CONFIG',
                data: {
                    wwObject: this.wwObject,
                }
            }

            try {
                const result = await wwLib.wwPopups.open(options);
                /*=============================================m_ÔÔ_m=============================================\
                  STYLE
                \================================================================================================*/
                if (typeof (result.markdownText) != 'undefined') {
                    this.wwObject.content.data.markdownText = result.markdownText;
                    this.wwObject.content.data.languageStyle = result.languageStyle;


                    this.wwObjectCtrl.update(this.wwObject)

                    hljs.initHighlighting()
                }

                // this.wwObjectCtrl.globalEdit(result);
                this.wwObjectCtrl.update(this.wwObject);

            } catch (error) {
                console.log(error);
            }
            wwLib.wwObjectHover.removeLock();
        }
    },
    mounted() {
        this.init();

        wwLib.wwElementsStyle.applyAllStyles({
            wwObject: this.wwObject,
            lastWwObject: null,
            element: this.$el,
            noAnim: this.wwAttrs.wwNoAnim,
            noClass: false,
        });

        this.$emit('ww-loaded', this);

    },
    beforeDestroyed() {
        //window.removeEventListener('resize', this.wwOnResize);
    }
};
</script>

<style lang="scss" scoped>
.ww-markdown {
    position: relative;
    width: 100%;
    height: 100%;
}

.ww-markdown-box {
    line-height: 1.4;

    border-radius: 6px;
    pointer-events: all;
}

.filler-object {
    min-height: 100px;
    display: block;
    overflow-x: auto;
    padding: 0.5em;
    color: #abb2bf;
    background: #282c34;
}

pre code.hljs.dockerfile::before,
pre code.hljs.css::before,
pre code.hljs.html::before,
pre code.hljs.javascript::before,
pre code.hljs.bash::before,
pre code.hljs.sql::before,
pre code.hljs.scss::before,
pre code.hljs.typescript::before,
pre code.hljs.http::before,
pre code.hljs.php::before {
    font-family: "Source Sans Pro", "Helvetica Neue", Arial, sans-serif;
    position: absolute;
    top: 0;
    right: 0;
    color: #ccc;
    text-align: right;
    font-size: 0.9em;
    padding: 5px 10px 0;
    line-height: 15px;
    height: 15px;
    font-weight: 600;
}
pre code.hljs.html::before {
    content: "HTML";
}
pre code.hljs.javascript::before {
    content: "JS";
}
pre code.hljs.bash::before {
    content: "Shell";
}
pre code.hljs.css::before {
    content: "CSS";
}
pre code.hljs.dockerfile::before {
    content: "Dockerfile";
}
pre code.hljs.sql::before {
    content: "SQL";
}
pre code.hljs.scss::before {
    content: "SCSS";
}
pre code.hljs.php::before {
    content: "PHP";
}
pre code.hljs.typescript::before {
    content: "Ts";
}
pre code.hljs.http::before {
    content: "HTTP";
}
@media (min-width: 768px) {
    .ww-markdown-box {
        padding: 2rem 2.5rem;
    }
}
</style>
