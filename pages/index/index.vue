<template>
	<view class="container" @tap="addParticles">
		<view class="text">{{ displayedText }}</view>
		<canvas canvas-id="loveCanvas" id="loveCanvas" class="canvas"></canvas>
	</view>
</template>

<script>
export default {
	data() {
		return {
			particles: [],
			numParticles: 80,
			canvasWidth: 0,
			canvasHeight: 0,
			backgroundColorIndex: 0,
			backgroundColors: ['#fce4ec', '#f8bbd0', '#f48fb1', '#f06292', '#ec407a', '#e91e63', '#ff80ab', '#ff4081', '#ff69b4'],
			scrollTexts: [],
			maxScrollTexts: 8,
			scrollTextSpeedRange: [1, 3],
			scrollTextList: [
				'我真的好喜欢你，像风走过了八千里。',
				'时光冗长，但我的心里只有你。',
				'愿与你共度每一个明天。',
				'你的笑容是我最大的动力。',
				'谢谢你走进我的世界。',
				'你是我藏在呼吸里的温柔。',
				'想和你一起等花开花落。',
				'你的名字是我写在星河里的诗。',
				'连晚霞都嫉妒我们的相遇。',
				'你眼中的光，照亮了我的四季。',
				'想把每个清晨煮成茶，慢慢陪你过完这一生。',
				'若三生有幸，只愿与你共赏同一轮明月。',
				'你心跳的频率，是我最安心的时钟。',
				'你的存在，让平凡的日子泛着微光。',
				'我穿越人海而来，只为触碰你指尖的温度。',
				'你是我的唯一依恋。',
				'与你在一起，每一天都是幸福。',
				'你的爱，是我一生中最美的遇见。',
				'希望我们的故事永不结束。',
				'感谢有你，让我成为更好的自己。',
				'你的笑容是我最美的回忆。',
				'每一刻与你相伴，都是生命的馈赠。',
				'你是我心中的阳光，驱散一切阴霾。',
				'与你同行，是我最明智的选择。',
				'你的温暖，让我心安。',
				'每一天都有你在身边，就是最好的日子。',
				'你是我生命中最亮的星。',
				'因为你，这个世界变得更加美好。',
				'你的陪伴，是我前行的动力。',
				'爱你，是我一生中最美好的决定。',
				'我这辈子就喜欢你！',
				'每一天都在期待与你相见。',
				'第一次和你牵手，我的心扑通扑通的~',
				'抱着你真的好温暖'
			],
			fullText: '我好喜欢你',
			displayedText: '',
			typingSpeed: 150
		};
	},
	onReady() {
		const { windowWidth, windowHeight } = uni.getSystemInfoSync();
		this.canvasWidth = windowWidth;
		this.canvasHeight = windowHeight;
		const ctx = uni.createCanvasContext('loveCanvas', this);
		this.particles = Array.from({ length: this.numParticles }, () => this.createParticle());
		setInterval(() => {
			this.backgroundColorIndex = (this.backgroundColorIndex + 1) % this.backgroundColors.length;
			this.changeBackgroundColor();
		}, 2000);
		this.addRandomScrollText();
		this.animate(ctx);
		this.startTypingEffect();
	},
	methods: {
		createParticle(x = null, y = null) {
			return {
				x: x ?? Math.random() * this.canvasWidth,
				y: y ?? Math.random() * this.canvasHeight,
				dx: (Math.random() - 0.5) * 1.5,
				dy: (Math.random() - 0.5) * 1.5,
				size: Math.random() * 6 + 6,
				opacity: Math.random() * 0.6 + 0.4
			};
		},
		drawHeart(ctx, x, y, size, color, opacity) {
			ctx.save();
			ctx.translate(x, y);
			ctx.scale(size / 10, size / 10);
			ctx.beginPath();
			ctx.moveTo(0, -3);
			ctx.bezierCurveTo(-5, -15, -15, -5, 0, 5);
			ctx.bezierCurveTo(15, -5, 5, -15, 0, -3);
			ctx.setFillStyle(color);
			ctx.setGlobalAlpha(opacity);
			ctx.fill();
			ctx.restore();
		},
		animate(ctx) {
			ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight);
			this.particles.forEach((p) => {
				p.x += p.dx;
				p.y += p.dy;
				if (p.x < 0 || p.x > this.canvasWidth) p.dx *= -1;
				if (p.y < 0 || p.y > this.canvasHeight) p.dy *= -1;
				this.drawHeart(ctx, p.x, p.y, p.size, '#ff69b4', p.opacity);
			});
			this.updateAndDrawScrollTexts(ctx);
			ctx.draw(false, () => requestAnimationFrame(() => this.animate(ctx)));
		},
		addParticles(event) {
			const { clientX, clientY } = event.touches[0];
			this.particles.push(...Array.from({ length: 10 }, () => this.createParticle(clientX, clientY)));
		},
		changeBackgroundColor() {
			const container = document.querySelector('.container');
			container && (container.style.backgroundColor = this.backgroundColors[this.backgroundColorIndex]);
		},
		addRandomScrollText() {
			if (this.scrollTexts.length >= this.maxScrollTexts) return;
			const text = this.scrollTextList[Math.floor(Math.random() * this.scrollTextList.length)];
			const y = Math.random() * this.canvasHeight;
			const speed = Math.random() * (this.scrollTextSpeedRange[1] - this.scrollTextSpeedRange[0]) + this.scrollTextSpeedRange[0];
			let x = this.canvasWidth;
			while (!this.isPositionAvailable(x, y, text)) x -= 10;
			this.scrollTexts.push({ text, x, y, speed });
			setTimeout(this.addRandomScrollText, 2000);
		},
		isPositionAvailable(x, y, text) {
			const ctx = uni.createCanvasContext('loveCanvas', this);
			ctx.setFontSize(20);
			const width = ctx.measureText(text).width;
			return !this.scrollTexts.some((t) => {
				const w = ctx.measureText(t.text).width;
				return x < t.x + w && x + width > t.x && y < t.y + 20 && y + 20 > t.y;
			});
		},
		updateAndDrawScrollTexts(ctx) {
			ctx.setFontSize(20);
			ctx.setFillStyle('#ffffff');
			this.scrollTexts = this.scrollTexts.filter((t) => {
				t.x -= t.speed;
				if (t.x + ctx.measureText(t.text).width < 0) return false;
				ctx.fillText(t.text, t.x, t.y);
				return true;
			});
		},
		startTypingEffect() {
			let index = 0;
			const id = setInterval(() => {
				this.displayedText = this.fullText.slice(0, index++);
				if (index > this.fullText.length) clearInterval(id);
			}, this.typingSpeed);
		}
	}
};
</script>

<style>
.container {
	width: 100vw;
	height: 100vh;
	position: relative;
	background-color: black;
	overflow: hidden;
	transition: background-color 2s ease-in-out;
}

.canvas {
	width: 100vw;
	height: 100vh;
	position: absolute;
	top: 0;
	left: 0;
}

.text {
	position: absolute;
	top: 50%;
	width: 100%;
	text-align: center;
	font-size: 36rpx;
	color: #fff;
	z-index: 10;
	font-weight: bold;
	letter-spacing: 2rpx;
	transform: translateY(-50%);
}
</style>
