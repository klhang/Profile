<template>
  <div id="app">
    <StyleEditor ref="styleEditor" :code="currentStyle"></StyleEditor>
    <ResumeEditor ref="resumeEditor" :markdown="currentMarkdown" :enableHtml="enableHtml"></ResumeEditor>
  </div>
</template>

<script>
  import StyleEditor from './components/StyleEditor'
  import ResumeEditor from './components/ResumeEditor'
  import './assets/reset.css'

  export default {
    name: 'app',
    components: {
      StyleEditor,
      ResumeEditor
    },
    data() {
      return {
        interval: 30,
        currentStyle: '',
        enableHtml: false,
        fullStyle: [
          `
/*
* Inspired by http://strml.net/
* Hi, I am Chris Guan.
* People are saying the summer hiring season is coming... I'd better build a website for myself.
* Alright, let me start with my resume.
*/

/* First, I need to add some animation on all of the elements */
* {
  transition: all .3s;
}
/* The white background is kind of boring, let's add some color */
html {
  color: rgb(222,222,222); background: rgb(0,43,54);
}
/* The texts seem to be too close to the border */
.styleEditor {
  padding: .5em;
  border: 1px solid;
  margin: 2em;
  overflow: auto;
  width: 45vw; height: 90vh;
}
/* And let's highlight the code */
.token.selector{ color: rgb(133,153,0); }
.token.property{ color: rgb(187,137,0); }
.token.punctuation{ color: yellow; }
.token.function{ color: rgb(42,161,152); }

/* How about some 3D animation */
html{
  perspective: 1000px;
}
.styleEditor {
  position: fixed; left: 0; top: 1;
  -webkit-transition: none;
  transition: none;
  -webkit-transform: rotateY(10deg) translateZ(-100px) ;
          transform: rotateY(10deg) translateZ(-100px) ;
}

/* Next, guess I need an editor */
.resumeEditor{
  position: fixed; right: 0; top: 0;
  padding: .5em;  margin: 1.5em;
  width: 48vw; height: 90vh;
  border: 1px solid;
  background: white; color: #222;
  overflow: auto;
}
/* All set, let me work on my resume */


`,
          `
/* Hmm... this looks weird
 * Oh，this is in Markdown，I want it to be more HR-friendly
 * Let me turn it into HTML
 */
`
          ,
          `
/* And add some styling on the HTML */
.resumeEditor{
  padding: 2em;
}
.resumeEditor h2{
  display: inline-block;
  border-bottom: 1px solid;
  margin: 1em 0 .5em;
}
.resumeEditor ul,.resumeEditor ol{
  list-style: none;
}
.resumeEditor ul> li::before{
  content: '•';
  margin-right: .5em;
}
.resumeEditor ol {
  counter-reset: section;
}
.resumeEditor ol li::before {
  counter-increment: section;
  content: counters(section, ".") " ";
  margin-right: .5em;
}
.resumeEditor blockquote {
  margin: 1em;
  padding: .5em;
  background: #ddd;
}
`],
        currentMarkdown: '',
        fullMarkdown: `Chris Guan
----
**Software Developer**


Skills
----
* **Language**：Java, JavaScript, Ruby, Python, TypeScript
* **Frontend Framework**: React, Angular2
* **Web App Framework**: Express, Django, Flask, RubyOnRails
* **Database**: MongoDB, Redis, PostgreSQL, SQLite3
* **DevOps**: Jenkins, AWS CodePipelin, EC2, IMA, Route53, CloudWatch
* **Others**: Docker, Nginx, RabbitMQ, Postman, NodeJS, Materialize, HTML, CSS, Git


Projects
----

* **CodeLab: A Collaborative Online Coding System.**  <a href="http://www.codelabsystems.com" target="_blank">LIVE</a>

Skills: TypeScript, Python, Angular2, Node.js, NodeJS, Redis, MongoDB, Docker, RESTful API, Nginx, Jenkins, AWS

---


* **Scape News: Real Time News Scraping and Filtering System.**  <a href="http://www.thescapenews.com" target="_blank">LIVE</a>

Skills: JavaScript, Java, Python, React, Express, Flask, Redis, MongoDB, RabbitMQ, TF-IDF, Jenkins, AWS

---


* **Curiosity: A Clone System of Quora.**  <a href="http://www.goodcuriosity.com" target="_blank">LIVE</a>

Skills: JavaScript, Ruby, React, Redux, Ruby on Rials, JQuery, RESTful API

---


* **Translate Overflow: A System for Sharing and Improving Translation Skills.**  <a href="https://translateoverflow.herokuapp.com" target="_blank">LIVE</a>

Skills: Python, React, Django, SQLite3, Materialize, RESTful API, Postman

---



Link
----
<a href="https://www.github.com/klhang" target="_blank">GitHub</a>

<a href="https://www.linkedin.com/in/chris-guan" target="_blank">Linkedin</a>

<a href="http://res.cloudinary.com/klhang/image/upload/v1532936393/ChrisGuan_resume.pdf" target="_blank">Click here to download Chris’s resume.</a>
`
      }
    },
    created() {
      this.makeResume()
    },

    methods: {
      makeResume: async function () {
        await this.progressivelyShowStyle(0)
        await this.progressivelyShowResume()
        await this.progressivelyShowStyle(1)
        await this.showHtml()
        await this.progressivelyShowStyle(2)
      },
      showHtml: function () {
        return new Promise((resolve, reject) => {
          this.enableHtml = true
          resolve()
        })
      },
      progressivelyShowStyle(n) {
        return new Promise((resolve, reject) => {
          let interval = this.interval
          let showStyle = (async function () {
            let style = this.fullStyle[n]
            if (!style) { return }
            // 计算前 n 个 style 的字符总数
            let length = this.fullStyle.filter((_, index) => index <= n).map((item) => item.length).reduce((p, c) => p + c, 0)
            let prefixLength = length - style.length
            if (this.currentStyle.length < length) {
              let l = this.currentStyle.length - prefixLength
              let char = style.substring(l, l + 1) || ' '
              this.currentStyle += char
              if (style.substring(l - 1, l) === '\n' && this.$refs.styleEditor) {
                this.$nextTick(() => {
                  this.$refs.styleEditor.goBottom()
                })
              }
              setTimeout(showStyle, interval)
            } else {
              resolve()
            }
          }).bind(this)
          showStyle()
        })
      },
      progressivelyShowResume() {
        return new Promise((resolve, reject) => {
          let length = this.fullMarkdown.length
          let interval = this.interval - 10
          let showResume = () => {
            if (this.currentMarkdown.length < length) {
              this.currentMarkdown = this.fullMarkdown.substring(0, this.currentMarkdown.length + 1)
              let lastChar = this.currentMarkdown[this.currentMarkdown.length - 1]
              let prevChar = this.currentMarkdown[this.currentMarkdown.length - 2]
              if (prevChar === '\n' && this.$refs.resumeEditor) {
                this.$nextTick(() => this.$refs.resumeEditor.goBottom())
              }
              setTimeout(showResume, interval)
            } else {
              resolve()
            }
          }
          showResume()
        })
      }
    }
  }

</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  html {
    min-height: 100vh;
  }
  *{
    box-sizing: border-box;
  }
</style>
