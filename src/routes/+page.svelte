<script>
	import { onMount } from 'svelte';

	let balls = [];
	let counter = 0;

	function createBall() {
		const size = Math.random() * 70 + 30; // Max size will be 100, min size will be 30
		return {
			x: Math.random() * (window.innerWidth - size), // Ensure ball fits horizontally
			y: Math.random() * (window.innerHeight - size), // Ensure ball fits vertically
			dx: (Math.random() - 0.5) * 4,
			dy: (Math.random() - 0.5) * 4,
			size: size
		};
	}

	function createBalls(num) {
		balls = [];
		for (let i = 0; i < num; i++) {
			balls.push(createBall());
		}
	}

	const updatePositions = () => {
		balls = balls.map((ball, index) => {
			let newBall = { ...ball };
			newBall.x += newBall.dx;
			newBall.y += newBall.dy;

			if (newBall.x + newBall.size >= window.innerWidth || newBall.x <= 0) {
				newBall.dx = -newBall.dx;
				counter++;
			}

			if (newBall.y + newBall.size >= window.innerHeight || newBall.y <= 0) {
				newBall.dy = -newBall.dy;
				counter++;
			}

			balls.forEach((otherBall, otherIndex) => {
				if (index !== otherIndex) {
					const dx = otherBall.x - newBall.x;
					const dy = otherBall.y - newBall.y;
					const distance = Math.sqrt(dx * dx + dy * dy);
					const minDistance = (newBall.size + otherBall.size) / 2;

					if (distance < minDistance) {
						const angle = Math.atan2(dy, dx);
						const speed1 = Math.sqrt(newBall.dx * newBall.dx + newBall.dy * newBall.dy);
						const speed2 = Math.sqrt(otherBall.dx * otherBall.dx + otherBall.dy * otherBall.dy);

						const direction1 = Math.atan2(newBall.dy, newBall.dx);
						const direction2 = Math.atan2(otherBall.dy, otherBall.dx);

						const newDx1 = speed1 * Math.cos(direction1 - angle);
						const newDy1 = speed1 * Math.sin(direction1 - angle);
						const newDx2 = speed2 * Math.cos(direction2 - angle);
						const newDy2 = speed2 * Math.sin(direction2 - angle);

						const finalDx1 =
							((newBall.size - otherBall.size) * newDx1 +
								(otherBall.size + otherBall.size) * newDx2) /
							(newBall.size + otherBall.size);
						const finalDy1 = newDy1;
						const finalDx2 =
							((otherBall.size - newBall.size) * newDx2 + (newBall.size + newBall.size) * newDx1) /
							(newBall.size + otherBall.size);
						const finalDy2 = newDy2;

						newBall.dx = Math.cos(angle) * finalDx1 + Math.cos(angle + Math.PI / 2) * finalDy1;
						newBall.dy = Math.sin(angle) * finalDx1 + Math.sin(angle + Math.PI / 2) * finalDy1;
						otherBall.dx = Math.cos(angle) * finalDx2 + Math.cos(angle + Math.PI / 2) * finalDy2;
						otherBall.dy = Math.sin(angle) * finalDx2 + Math.sin(angle + Math.PI / 2) * finalDy2;

						counter++;
					}
				}
			});

			return newBall;
		});
	};

	const animate = () => {
		updatePositions();
		requestAnimationFrame(animate);
	};

	onMount(() => {
		createBalls(10);
		animate();
	});
</script>

<div class="min-h-screen">
	<div class="fixed left-5 top-5 text-xl font-bold text-black">
		Hits and Collisions: {counter}
	</div>

	{#each balls as ball}
		<div
			class="absolute rounded-full bg-red-500"
			style="left: {ball.x}px; top: {ball.y}px; width: {ball.size}px; height: {ball.size}px;"
		></div>
	{/each}
</div>
