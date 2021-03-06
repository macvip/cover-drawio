<template>
  <Loading v-model:active="loading" />
  <div id="content" v-if="!loading" >
    <template v-if="svgWork">
    <div id="toolmenu">
      <CicleMenu />
    </div>
    <div class="imgFrame">
        <img id="svgImage" :src="'/' + svgWork" />
    </div>
    </template>
    <template v-else>
    <div class="newdraw">
      <button class="drawBtn" @click="openDrawio">
          <img style="width: 32px;" src="images/sketch.png" />
          <span>  新建绘图 </span>
      </button>
      <p>在 Drawio 挂件中保存绘图后，请点击下方刷新按钮</p>
      <button @click="reloadAll">
          <img class="btnIcon" src="images/toolbar/refresh.png" />
          <span>刷新</span>
      </button>
    </div>
    </template>
  </div>
</template>

<script setup>
import { ref, provide, onBeforeMount } from 'vue'
import CicleMenu from './components/ciclemenu'
import { copyToClipboard } from './utils/clipboard'
import Loading from 'vue-loading-overlay'
import 'vue-loading-overlay/dist/vue-loading.css'

const blockId = ref(null)
const loading = ref(true)
const svgWork = ref(null)

const parentDom = window.frameElement.parentElement.parentElement

blockId.value = parentDom.getAttribute('data-node-id')

fetch('/api/attr/getBlockAttrs', {
    body: JSON.stringify({
        id: blockId.value,
    }),
    method: 'POST',
}).then(response => {
    return response.json()
}).then(data => {
    let asset = data.data['custom-data-assets'] || data.data['data-assets']
    if(asset) {
        svgWork.value = asset
    }
    loading.value = false
    resetAsset()
})

var resizeTimer = null;

function resetAsset() {
    if(!svgWork.value)
        return
    if(resizeTimer)
        clearTimeout(resizeTimer)
    resizeTimer = setTimeout(() => {
        setAsset(svgWork.value)
    }, 2000)
}

window.addEventListener('resize', resetAsset, false)

function openDrawio() {
    window.open('/widgets/drawio/?id=' + blockId.value, '_blank')
}

function selfReload() {
    if(svgWork.value) {
        let svg = document.querySelector('#svgImage')
        svg.src = '/' + svgWork.value + '?t=' + (new Date().getTime())
    }
}

function reloadAll() {
    window.location.reload()
}

function setAsset(asset) {
    fetch('/api/attr/setBlockAttrs', {
        body: JSON.stringify({
            id: blockId.value,
            attrs: {
                'data-assets': asset,
                'custom-data-assets': asset
            }
        }),
        method: 'POST',
    })//.then(response => {
    //    return response.json()
    //})
}

function openPreview() {
    let svgSrc = document.querySelector('#svgImage').src
    window.parent.document.previewDraw(svgSrc)
}

function savePng() {
    fetch('/' + svgWork.value).then(response => {
        return response.text()
    }).then(data => {
        var image = new Image()
        image.onload = () => {
            const width = image.width
            const height = image.height
            const canvas = document.createElement('canvas')
            canvas.width = width * 2 
            canvas.height = height * 2
            canvas.style.width = width + 'px'
            canvas.style.height = height + 'px'

            const context = canvas.getContext('2d')
            context.scale(2, 2)
            context.drawImage(image, 0, 0)

            let dataURL = canvas.toDataURL('image/png')
            let a = document.createElement('a')
            a.href = dataURL
            let _ = svgWork.value
            let filename = _.substring('assets/'.length, _.lastIndexOf('.'))
            a.download = filename
            a.click()
        }
        image.src = 'data:image/svg+xml;base64,' + window.btoa(unescape(encodeURIComponent(data)))
    })
}

function copyMarkdownLink() {
    let filename = svgWork.value.substring('assets/'.length)
    let mdLink = `![${filename}](${svgWork.value})`
    if (navigator.clipboard) 
        navigator.clipboard.writeText(mdLink)
    else
        copyToClipboard(mdLink)
}

function itemClicked(name) {
    switch(name) {
        case 'Preview':
            openPreview()
            break
        case 'Refresh':
            selfReload()
            break
        case 'Edit':
            openDrawio()
            break
        case 'Save':
            savePng()
            break
        case 'Copy':
            copyMarkdownLink()
            break
        default:
            break
    }
}
    
provide('itemClicked', itemClicked)

onBeforeMount(()=>{
    let root = window.parent.document
    if (!root.getElementById('protyleViewerScript')) {
        let vs = root.createElement('script')
        vs.id = 'protyleViewerScript'
        vs.async = true
        vs.src = '/stage/protyle/js/viewerjs/viewer.js?v=1.10.4'
        root.head.appendChild(vs)
    }
    if (!root.getElementById('previewdraw')) {
        let vv = root.createElement('script')
        vv.id = 'previewdraw'
        vv.innerHTML = `
            function previewDraw(src) {
                let img = document.createElement("img");
                img.src = src;
                const pv = new Viewer(img,{
                    button: false,
                    hidden: function() {
                        pv.destroy()
                    },
                    toolbar: {
                        zoomIn: true,
                        zoomOut: true,
                        oneToOne: true,
                        reset: true,
                        prev: true,
                        play: true,
                        next: true,
                        rotateLeft: true,
                        rotateRight: true,
                        flipHorizontal: true,
                        flipVertical: true,
                        close: function() {
                            pv.destroy()
                        }
                    }
                });
                pv.show();
            }
            document.previewDraw = previewDraw;     
        `
        root.head.appendChild(vv)
    }    
})

</script>


<style scoped>
#content {
  display: flex;
  position: absolute;
  bottom: 1vh;
  left: 1vw;
  width: 98vw;
  height: 98vh;
  align-items: center;
  justify-content: center;
}

#toolmenu {
    display: none;
    position: fixed;
    top: 50px;
    right: 40px;
    transform: translate(-50%, -50%);
    z-index: 1;
}

#content:hover #toolmenu {
  display: block;
}

.btnIcon {
  width: 32px;
}

button {
  align-items: center;
  display: flex;
}

.imgFrame {
  display: flex;
}

#svgImage {
  max-width: 100%;
}

button:hover .btnIcon {
  filter: invert(18%) sepia(48%) saturate(4268%) hue-rotate(348deg) brightness(91%) contrast(81%);
}

button:hover span {
  color: brown;
}

.drawBtn {
  margin: 0 auto;
}

.newdraw {
  height: 100vh;
  display: grid;
  justify-items: center;  
  align-content: center;
}

</style>
