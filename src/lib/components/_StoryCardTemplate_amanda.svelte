<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	const data = [
		{ borough: 'Bronx',         year2017: 30.7, year2021: 35.4 },
		{ borough: 'Brooklyn',      year2017: 26.7, year2021: 26.3 },
		{ borough: 'Manhattan',     year2017: 22.4, year2021: 19.5 },
		{ borough: 'Queens',        year2017: 18.8, year2021: 19.7 },
		{ borough: 'Staten Island', year2017: 12.3, year2021: 11.8 }
	].sort((a, b) => b.year2021 - a.year2021);

	const subgroups = ['year2017', 'year2021'];
	const colors    = { year2017: '#c9d9e8', year2021: '#1a5fa8' };
	const labels    = { year2017: '2017',    year2021: '2021'    };

	let svgEl;
	let tooltip = $state({ visible: false, x: 0, y: 0, text: '' });

	onMount(() => {
		const margin = { top: 30, right: 60, bottom: 50, left: 110 };
		const totalHeight = 375;
		const width  = svgEl.clientWidth - margin.left - margin.right;
		const height = totalHeight - margin.top - margin.bottom;

		const svg = d3.select(svgEl)
			.attr('height', totalHeight)
			.append('g')
			.attr('transform', `translate(${margin.left},${margin.top})`);

		const boroughs = data.map(d => d.borough);
		const y0 = d3.scaleBand().domain(boroughs).range([0, height]).padding(0.3);
		const y1 = d3.scaleBand().domain(subgroups).range([0, y0.bandwidth()]).padding(0.08);
		const x  = d3.scaleLinear().domain([0, 42]).range([0, width]);

		// Gridlines + top axis
		svg.append('g')
			.call(d3.axisTop(x).ticks(6).tickSize(-height).tickFormat(d => d + '%'))
			.call(g => g.select('.domain').remove())
			.call(g => g.selectAll('.tick line').attr('stroke', '#e0e0e0').attr('stroke-dasharray', '3,3'))
			.call(g => g.selectAll('.tick text').attr('fill', '#999').attr('font-size', '11px'));

		// Borough labels
		svg.append('g')
			.call(d3.axisLeft(y0).tickSize(0))
			.call(g => g.select('.domain').remove())
			.call(g => g.selectAll('.tick text').attr('font-size', '13px').attr('fill', '#222').attr('dx', '-8'));

		// Bar groups
		const groups = svg.selectAll('.borough-group')
			.data(data)
			.join('g')
			.attr('class', 'borough-group')
			.attr('transform', d => `translate(0, ${y0(d.borough)})`);

		groups.selectAll('rect')
			.data(d => subgroups.map(key => ({ key, value: d[key], borough: d.borough })))
			.join('rect')
			.attr('y', d => y1(d.key))
			.attr('height', y1.bandwidth())
			.attr('x', 0)
			.attr('width', 0)
			.attr('rx', 2)
			.attr('fill', d => colors[d.key])
			.on('mouseover', function (event, d) {
				d3.select(this).attr('opacity', 0.8);
				const rect = svgEl.getBoundingClientRect();
				tooltip = {
					visible: true,
					x: event.clientX - rect.left,
					y: event.clientY - rect.top - 10,
					text: `${d.borough}, ${labels[d.key]}: ${d.value}%`
				};
			})
			.on('mousemove', function (event) {
				const rect = svgEl.getBoundingClientRect();
				tooltip.x = event.clientX - rect.left;
				tooltip.y = event.clientY - rect.top - 10;
			})
			.on('mouseout', function () {
				d3.select(this).attr('opacity', 1);
				tooltip.visible = false;
			})
			.transition().duration(700).ease(d3.easeCubicOut).delay((_, i) => i * 60)
			.attr('width', d => x(d.value));

		// Value labels
		groups.selectAll('.label')
			.data(d => subgroups.map(key => ({ key, value: d[key] })))
			.join('text')
			.attr('class', 'label')
			.attr('y', d => y1(d.key) + y1.bandwidth() / 2 + 4)
			.attr('x', d => x(d.value) + 5)
			.attr('font-size', '11px')
			.attr('fill', d => d.key === 'year2021' ? '#1a5fa8' : '#999')
			.attr('font-weight', d => d.key === 'year2021' ? '600' : '400')
			.text(d => d.value + '%')
			.attr('opacity', 0)
			.transition().duration(700).delay((_, i) => i * 60 + 400)
			.attr('opacity', 1);

		// Legend
		const legend = svg.append('g').attr('transform', `translate(0, ${height + 14})`);
		Object.entries(labels).forEach(([key, label], i) => {
			const g = legend.append('g').attr('transform', `translate(${i * 80}, 0)`);
			g.append('rect').attr('width', 12).attr('height', 12).attr('rx', 2).attr('fill', colors[key]);
			g.append('text').attr('x', 17).attr('y', 10).attr('font-size', '12px').attr('fill', '#555').text(label);
		});
	});
</script>

<article class="story-card">
	<hr class="divider" />

	<h2 class="headline">One in Three Bronx Children Live With Mice</h2>
	<p class="byline">By Amanda</p>

	<p>Across New York City, nearly one in four children ages 1 to 13 live in homes with mice — but the burden falls hardest in the Bronx, where that share has risen to 35% in 2021.</p>
	<p>Staten Island children are the least exposed at 12%, while the Bronx figure is three times higher, reflecting deep inequalities in housing quality across the city.</p>

	<figure class="chart">
		<figcaption class="chart-title">% of children ages 1–13 living in homes with mice, by borough</figcaption>

		<div class="chart-workspace">
			<div class="svg-wrap">
				<svg bind:this={svgEl}></svg>
				{#if tooltip.visible}
					<div class="tooltip" style="left:{tooltip.x}px; top:{tooltip.y}px">
						{tooltip.text}
					</div>
				{/if}
			</div>
		</div>

		<figcaption class="chart-footer">Source: NYC DOHMH Environment & Health Data Portal, NYC Kids Survey (2017, 2021)</figcaption>
	</figure>
</article>

<style>
	.story-card {
		margin-bottom: 3rem;
	}

	.headline {
		font-size: 1.75rem;
		font-weight: 700;
		line-height: 1.2;
		margin: 0 0 0.5rem;
	}

	.byline {
		font-size: 0.875rem;
		color: #888;
		margin: 0 0 1.5rem;
		text-transform: uppercase;
		letter-spacing: 0.04em;
	}

	p {
		font-size: 1.0625rem;
		line-height: 1.7;
		margin: 0 0 1rem;
		max-width: 640px;
	}

	.chart {
		margin: 2rem 0 0;
	}

	.chart-title {
		font-size: 0.9rem;
		font-weight: 600;
		color: #222;
		margin-bottom: 1rem;
		display: block;
	}

	.chart-workspace {
		background: #fff;
		padding: 1rem 0.5rem 0.5rem;
	}

	.svg-wrap {
		position: relative;
	}

	svg {
		display: block;
		width: 100%;
	}

	.tooltip {
		position: absolute;
		background: #222;
		color: #fff;
		font-size: 12px;
		padding: 5px 10px;
		border-radius: 4px;
		pointer-events: none;
		white-space: nowrap;
		transform: translateX(-50%);
	}

	.chart-footer {
		margin-top: 0.75rem;
		font-size: 0.8rem;
		color: #999;
		display: block;
	}
</style>
