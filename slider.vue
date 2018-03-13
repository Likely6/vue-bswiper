<template>
	<div class="slider" ref="slider">
		<div class="slider-wrapper" ref="sliderWrapper">
			<slot></slot>
		</div>
		<div class="dots" v-if="showDot">
			<span class="dot" v-for="(item, index) in dots" :key="index" :class="{active: currentPageIndex === index}"></span>
		</div>
	</div>
</template>
<script>
import BScroll from 'better-scroll'
import {addClass} from './func.js'
export default {
	props: {
		loop: {
			type: Boolean,
			default: true
		},
		autoPlay: {
			type: Boolean,
			default: true
		},

		speed: {
			type: Number,
			default: 800
		},
		interval: {
			type: Number,
			default: 2000
		},
		showDot: {
			type: Boolean,
			default: true
		}
	},
	data() {
		return {
			attr: {
				scrollX: true,
				scrollY: false,
				momentum: false,
				click: true,
				probeType: 3,
				snap: {
					loop: this.loop,
					threshold: 0.1,
					speed: this.speed
				}
			},
			dots: [],
			currentPageIndex: 0
		}
	},
	methods: {
		_setWidth(isResize) {
			this.children = this.$refs.sliderWrapper.children

			let width = 0
			let sliderWidth = this.$refs.slider.clientWidth
			for (let i = 0; i < this.children.length; i++) {
				let child = this.children[i]
				addClass(child, 'slider-item')
				child.style.width = sliderWidth + 'px'
				width += sliderWidth
			}
			if (this.loop && !isResize) {
				width += 2 * sliderWidth
			}
			this.$refs.sliderWrapper.style.width = width + 'px'
		},
		_initSlider() {
			this.slider = new BScroll(this.$refs.slider, this.attr)
			this.slider.on('scrollEnd', () => {
				let pageIndex = this.slider.getCurrentPage().pageX
				this.currentPageIndex = pageIndex
				if (this.autoPlay) {
					this._play()
				}
			})
		},
		_initDots() {
			this.dots = new Array(this.children.length)
		},
		_play() {
			clearTimeout(this.timer)
			this.timer = setTimeout(() => {
				this.slider.next()
			}, this.interval)
		},
		prev() {
        	this.slider.prev()
      	},
		next() {
			this.slider.next()
		}
	},
	mounted() {
		this._setWidth()
		this._initDots()
		this._initSlider()
		if (this.autoPlay) {
			this._play()
		}
		window.addEventListener('resize', () => {
			if (!this.slider) {
				return
			}
			this._setWidth(true)
			this.slider.refresh()
		})
	},
	deactivated() {
    	this.slider.disable()
    	clearTimeout(this.timer)
    },
    beforeDestroy() {
    	this.slider.disable()
    	clearTimeout(this.timer)
    }
}
</script>
<style lang="scss" scoped>
.slider{
	position: relative;
	width: 100%;
	height: 100%;
	overflow: hidden;
	.slider-wrapper{
		height: 100%;
		white-space: nowrap;
		.slider-item{
			float: left;
			display: inline-block;
			height: 100%;
			img{
				width: 100%;
				height: 100%;
			}
		}
	}
	.dots{
    	position: absolute;
    	right: 0;
    	left: 0;
    	bottom: 12px;
    	text-align: center;
    	font-size: 0;
    	.dot{
        	display: inline-block;
        	margin: 0 4px;
        	width: 8px;
        	height: 8px;
        	border-radius: 50%;
        	background: #ccc;
        	&.active{
        		width: 20px;
        		border-radius: 5px;
        		background: #fff;
        	}
        }
    }
}
</style>