<template>
    <div>
        <div :class="`bubble ${bubbleClass}`">
            <div><em class="sender">{{ sender }} says:</em><span class="time">{{ time }}</span></div>
            <h2>{{ message }}</h2>
        </div>
    </div>
</template>
<style lang="scss" scoped>

    .bubble {
        box-sizing: border-box;
        float: left;
        width: auto;
        max-width: 80%;
        position: relative;
        clear: both;
        
        background-image: -webkit-gradient(linear, left bottom, left top, color-stop(0.15, #bee2ff), color-stop(1, #95c2fd));
        background-image: -webkit-linear-gradient(bottom, #bee2ff 15%, #95c2fd 100%);
        background-image: -moz-linear-gradient(bottom, #bee2ff 15%, #95c2fd 100%);
        background-image: -ms-linear-gradient(bottom, #bee2ff 15%, #95c2fd 100%);
        background-image: -o-linear-gradient(bottom, #bee2ff 15%, #95c2fd 100%);
        background-image: linear-gradient(bottom, #bee2ff 15%, #95c2fd 100%);
        border: solid 1px rgba(0,0,0,0.5);
        -webkit-border-radius: 20px;
        -moz-border-radius: 20px;
        border-radius: 20px;

        -webkit-box-shadow: inset 0 8px 5px rgba(255,255,255,0.65), 0 1px 2px rgba(0,0,0,0.2);
        -moz-box-shadow: inset 0 8px 5px rgba(255,255,255,0.65), 0 1px 2px rgba(0,0,0,0.2);
        box-shadow: inset 0 8px 5px rgba(255,255,255,0.65), 0 1px 2px rgba(0,0,0,0.2);
        
        margin-bottom: 20px;
        padding: 6px 20px;
        color: #000;
        text-shadow: 0 1px 1px rgba(255,255,255,0.8);
        word-wrap: break-word;
    }

    .outgoing {
        background: #a1d2f3;
    }

    .incoming {
        background: rgb(150, 252, 181);
    }
</style>
<script>
export default {
    props: ['message', 'sender', 'time', 'outgoing'],
    computed: {
        bubbleClass() {
            return this.outgoing ? 'outgoing' : 'incoming';
        }
    }
}
</script>