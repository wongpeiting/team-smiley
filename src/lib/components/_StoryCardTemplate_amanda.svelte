<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	const data = [
		{ borough: 'Bronx',        pct2017: 30.7, pct2021: 35.4 },
		{ borough: 'Brooklyn',     pct2017: 26.7, pct2021: 26.3 },
		{ borough: 'Manhattan',    pct2017: 22.4, pct2021: 19.5 },
		{ borough: 'Queens',       pct2017: 18.8, pct2021: 19.7 },
		{ borough: 'Staten Island',pct2017: 12.3, pct2021: 11.8 }
	].sort((a, b) => b.pct2021 - a.pct2021);

	const COLORS = { pct2017: '#c9d9e8', pct2021: '#1a5fa8' };
	const YEARS  = { pct2017: '2017',    pct2021: '2021'    };

	let svgEl;
	let tip = $state({ show: false, text: '', x: 0, y: 0 });

	onMount(() => {
		const m = { top: 30, right: 55, bottom: 50, left: 110 };
		const W = svgEl.clientWidth - m.left - m.right;
		const H = 360 - m.top - m.bottom;

		const svg = d3.select(svgEl)
			.attr('height', 360)
			.append('g')
			.attr('transform', `translate(${m.left},${m.top})`);

		const boroughs  = data.map(d => d.borough);
		const keys      = ['pct2017', 'pct2021'];

		const y0 = d3.scaleBand().domain(boroughs).range([0, H]).padding(0.25);
		const y1 = d3.scaleBand().domain(keys).range([0, y0.bandwidth()]).padding(0.08);
		const x  = d3.scaleLinear().domain([0, 42]).range([0, W]);

		// gridlines + top axis
		svg.append('g')
			.call(d3.axisTop(x).ticks(5).tickFormat(d => d + '%').tickSize(-H))
			.call(g => g.select('.domain').remove())
			.call(g => g.selectAll('.tick line').attr('stroke', '#e8e8e8'))
			.call(g => g.selectAll('.tick text').attr('fill', '#aaa').attr('font-size', 11));

		// borough labels
		svg.append('g')
			.call(d3.axisLeft(y0).tickSize(0))
			.call(g => g.select('.domain').remove())
			.call(g => g.selectAll('.tick text').attr('font-size', 13).attr('fill', '#222').attr('dx', -8));

		// bars
		const groups = svg.selectAll('g.group')
			.data(data)
			.join('g')
			.attr('class', 'group')
			.attr('transform', d => `translate(0,${y0(d.borough)})`);

		groups.selectAll('rect')
			.data(d => keys.map(k => ({ k, v: d[k], borough: d.borough })))
			.join('rect')
			.attr('y',      d => y1(d.k))
			.attr('height', y1.bandwidth())
			.attr('rx', 2)
			.attr('fill', d => COLORS[d.k])
			.attr('width', 0)
			.on('mouseover', function (e, d) {
				d3.select(this).attr('opacity', 0.75);
				const r = svgEl.getBoundingClientRect();
				tip = { show: true, text: `${d.borough} ${YEARS[d.k]}: ${d.v}%`, x: e.clientX - r.left, y: e.clientY - r.top - 12 };
			})
			.on('mousemove', function (e) {
				const r = svgEl.getBoundingClientRect();
				tip.x = e.clientX - r.left;
				tip.y = e.clientY - r.top - 12;
			})
			.on('mouseout', function () {
				d3.select(this).attr('opacity', 1);
				tip.show = false;
			})
			.transition().duration(650).ease(d3.easeCubicOut).delay((_, i) => i * 50)
			.attr('width', d => x(d.v));

		// value labels
		groups.selectAll('text.val')
			.data(d => keys.map(k => ({ k, v: d[k] })))
			.join('text')
			.attr('class', 'val')
			.attr('y',    d => y1(d.k) + y1.bandwidth() / 2 + 4)
			.attr('fill', d => d.k === 'pct2021' ? '#1a5fa8' : '#aaa')
			.attr('font-size', 11)
			.attr('font-weight', d => d.k === 'pct2021' ? 600 : 400)
			.text(d => d.v + '%')
			.attr('opacity', 0)
			.transition().duration(400).delay((_, i) => i * 50 + 500)
			.attr('x',      d => x(d.v) + 5)
			.attr('opacity', 1);

		// legend
		const leg = svg.append('g').attr('transform', `translate(0,${H + 16})`);
		keys.forEach((k, i) => {
			const g = leg.append('g').attr('transform', `translate(${i * 72},0)`);
			g.append('rect').attr('width', 11).attr('height', 11).attr('rx', 2).attr('fill', COLORS[k]);
			g.append('text').attr('x', 15).attr('y', 9.5).attr('font-size', 12).attr('fill', '#666').text(YEARS[k]);
		});
	});
</script>

<article class="story-card">
	<h2 class="headline">One in Three Bronx Children Live With Mice</h2>
	<p class="byline">By Amanda</p>

	<p>Across New York City, nearly one in four children ages 1 to 13 live in homes with mice — but the burden falls hardest in the Bronx, where that share has risen to 35% in 2021.</p>
	<p>Staten Island children are the least exposed at 12%, roughly three times lower than the Bronx, reflecting deep inequalities in housing quality across the city.</p>

	<figure class="chart">
		<figcaption class="chart-title">% of children ages 1–13 living in homes with mice, by borough</figcaption>
		<div class="chart-wrap">
			<svg bind:this={svgEl}></svg>
			{#if tip.show}
				<div class="tip" style="left:{tip.x}px; top:{tip.y}px">{tip.text}</div>
			{/if}
		</div>
		<figcaption class="chart-footer">Source: NYC DOHMH Environment & Health Data Portal, NYC Kids Survey (2017, 2021)</figcaption>
	</figure>
</article>

<style>
	.story-card  { margin-bottom: 3rem; }

	.headline {
		font-size: 1.75rem;
		font-weight: 700;
		line-height: 1.2;
		margin: 0 0 0.5rem;
	}

	.byline {
		font-size: 0.8rem;
		color: #999;
		text-transform: uppercase;
		letter-spacing: 0.06em;
		margin: 0 0 1.5rem;
	}

	p {
		font-size: 1rem;
		line-height: 1.7;
		margin: 0 0 1rem;
		max-width: 600px;
	}

	.chart       { margin: 2rem 0 0; }

	.chart-title {
		display: block;
		font-size: 0.875rem;
		font-weight: 600;
		margin-bottom: 1rem;
	}

	.chart-wrap  { position: relative; }

	svg {
		display: block;
		width: 100%;
	}

	.tip {
		position: absolute;
		background: #222;
		color: #fff;
		font-size: 12px;
		padding: 4px 10px;
		border-radius: 4px;
		pointer-events: none;
		white-space: nowrap;
		transform: translateX(-50%);
	}

	.chart-footer {
		display: block;
		margin-top: 0.75rem;
		font-size: 0.78rem;
		color: #aaa;
	}
</style>
