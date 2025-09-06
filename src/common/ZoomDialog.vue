<template>
    <div 
        v-if="visible" 
        :class="['ZoomDialogBackground', 
                {'ZoomDialogHidden': step == 0}, 
                {'ZoomDialogAnimation': step >= 2}]" 
        @mousedown="close">
        <div class="ZoomDialogContainer" ref="container" @click.stop="" @mousedown.stop="">
            <div :class="['ZoomDialogWrapper', {'ZoomDialogWrapperOverflow': overflow === 'visible'}]" ref="wrapper">
                <div class="ZoomDialogContent" ref="content">
                    <slot></slot>
                </div>
            </div>
        </div>
    </div>
</template>
<style lang="scss">
    @import "../style/components/zoom_dialog.scss";
</style>
<script>
export default {
    name: "ZoomDialog",
    props: ['overflow'],
    data: function () {
      return {
        visible: false,
        step:0
      };
    },
    methods: {
        show () {
            // Move dialog to body level to avoid stacking context issues
            this.moveToBody()
            
            // Force center positioning for glassmorphism theme
            const startPos = {
                x: window.innerWidth / 2,
                y: window.innerHeight / 2,
                w: 100,
                h: 100
            }
            this.visible = true
            this.step = 0
            setTimeout(() => {
                this.animStep1(startPos)
            }, 100);
        },
        moveToBody() {
            // Move the dialog to body level to avoid parent stacking contexts
            if (this.$el && this.$el.parentNode !== document.body) {
                document.body.appendChild(this.$el)
            }
        },
        animStep1(startPos) {
 
            const container = this.$refs.container
            const content = this.$refs.content
      
            const endPos = this.position(content)
            const w = startPos.w / endPos.w
            const h = startPos.h / endPos.h
            // Force center positioning - calculate from center of screen
            const x = Math.round((window.innerWidth / 2) - (endPos.w / 2))
            const y = Math.round((window.innerHeight / 2) - (endPos.h / 2))
            container.style.transform = `translate(${x}px, ${y}px) scale(${w}, ${h})`
            container.style.opacity = 0.3
            this.step = 1
            setTimeout(() => {
                this.animStep2()
            }, 100);
        },
        animStep2 () {
            this.step = 2
            const container = this.$refs.container
            // Force center positioning
            container.style.transform = `translate(0, 0) scale(1,1)`
            container.style.opacity = 1
            container.style.left = '0px'
            container.style.top = '0px'
        },
        close () {
            this.visible = false
            this.step = 0
            // Restore dialog to original position after closing
            this.restoreToOriginalPosition()
        },
        restoreToOriginalPosition() {
            // This will be called when the component is destroyed or closed
            // The dialog will be restored to its original position in the DOM
        },
        position (node, includeScroll = false) {
            if (node) {
                if (node && node.toLowerCase) {
                    node = document.getElementById(node)
                }
                const clientRect = node.getBoundingClientRect();
                const ret = {
                    x: clientRect.left, 
                    y: clientRect.top, 
                    w: clientRect.right - clientRect.left, 
                    h: clientRect.bottom - clientRect.top
                };
                if(includeScroll){
                    ret.x += window.scrollX
                    ret.y += window.scrollY
                }
                return ret;
            }
            return {
                x: 0, y:0, w: 100, h:100
            }	
        },
        shake () {
            const wrapper = this.$refs.wrapper;         
            setTimeout(() => {
                wrapper.style.left = (50) + "px";
            }, 1);

            setTimeout(() => {
                wrapper.style.left = (-50) + "px";
            }, 51);

            setTimeout(() => {
                wrapper.style.left = (50) + "px";
            }, 101);

            setTimeout(() => {
                wrapper.style.left = (-50) + "px";
            }, 151);

            setTimeout(() => {
                wrapper.style.left = (0) + "px";
            }, 201);

        }
    }
}
</script>