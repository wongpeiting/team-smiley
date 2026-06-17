<script>
	import * as d3 from 'd3'
	import rawData from '$lib/data/rat-hotspots.json'
	import locData from '$lib/data/rat-locations.json'

	// 1. data — sum Jan-May counts per year for each address
	const data = rawData.map(d => {
		const years = [2023, 2024, 2025, 2026]
		const yearTotals = years.map(y => ({
			year: y,
			count: d.months
				.filter(m => m.year === y)
				.reduce((sum, m) => sum + m.count, 0)
		}))
		return {
			address: d.address,
			borough: d.borough,
			change: d.change,
			type: d.type,
			yearTotals
		}
	})

	// 2. dimensions — for each small multiple
	const width = 260
	const height = 200
	const margin = { top: 50, right: 10, bottom: 25, left: 40 }

	// 3. scales
	// x: years are categories → scaleBand
	const xScale = d3.scaleBand()
		.domain([2023, 2024, 2025, 2026])
		.range([margin.left, width - margin.right])
		.padding(0.2)

	// y: separate scales for drops and spikes
	const dropMax = d3.max(data.filter(d => d.type === 'drop'), d => d3.max(d.yearTotals, t => t.count))
	const yScaleDrop = d3.scaleLinear()
		.domain([0, dropMax])
		.range([height - margin.bottom, margin.top])

	const yScaleSpike = d3.scaleLinear()
		.domain([0, 200])
		.range([height - margin.bottom, margin.top])

	// color: drop vs spike → scaleOrdinal
	const colorScale = d3.scaleOrdinal()
		.domain(["drop", "spike"])
		.range(["#4e79a7", "#e15759"])

	// --- Chart 2: location breakdown ---
	const locWidth = 500
	const locHeight = 200
	const locMargin = { top: 10, right: 60, bottom: 20, left: 120 }

	const locYScale = d3.scaleBand()
		.domain(locData.map(d => d.category))
		.range([locMargin.top, locHeight - locMargin.bottom])
		.padding(0.3)

	const locXScale = d3.scaleLinear()
		.domain([0, d3.max(locData, d => d.count)])
		.range([locMargin.left, locWidth - locMargin.right])

	const locColorScale = d3.scaleOrdinal()
		.domain(["Homes", "Offices", "Public spaces", "Other"])
		.range(["#8b5e3c", "#8b5e3c", "#8b5e3c", "#8b5e3c"])

	const fmt = d3.format(",")
</script>

<article class="story-card">
	<hr class="divider" />

	<h2 class="headline">Two in three rodent complaints come from homes</h2>
	<p class="byline">By Pei Ting Wong</p>

	<p>The vast majority of New Yorkers reporting rodent problems to 311 are doing so from where they live — not where they work or play.</p>

	<figure class="chart">
		<figcaption class="chart-title">Rodent complaints by location type (2020–2026)</figcaption>

		<div class="chart-workspace">
			<svg viewBox="0 0 {locWidth} {locHeight}" width="100%">
				{#each locData as d}
					<rect
						x={locMargin.left}
						y={locYScale(d.category)}
						width={locXScale(d.count) - locMargin.left}
						height={locYScale.bandwidth()}
						fill={locColorScale(d.category)}
					/>
					<text
						x={locMargin.left - 8}
						y={locYScale(d.category) + locYScale.bandwidth() / 2 + 4}
						text-anchor="end"
						class="cat-label"
					>
						{d.category}
					</text>
					<text
						x={locXScale(d.count) + 5}
						y={locYScale(d.category) + locYScale.bandwidth() / 2 + 4}
						class="count-label"
					>
						{fmt(d.count)}
					</text>
				{/each}
			</svg>
		</div>

		<figcaption class="chart-footer">Source: NYC 311 Service Requests (Open Data)</figcaption>
	</figure>
</article>

<article class="story-card">
	<hr class="divider" />

	<h2 class="headline">What silenced the rats at Wagner Houses?</h2>
	<p class="byline">By Pei Ting Wong</p>

	<p>For years, residents of 2400 2nd Avenue in East Harlem — part of the Wagner Houses, a NYCHA complex with a long, documented history of severe rodent infestations — filed hundreds of 311 complaints about rats running through apartments and building grounds. Then, in 2026, the complaints stopped entirely. What changed? NYCHA has not said. Did a remediation effort finally work, or did tenants simply give up reporting?</p>
	<p>But as one rat hotspot goes quiet, another is brewing. Over at 2337 Grand Concourse in the Bronx, complaints have surged from near zero to dozens in just two years — a trajectory that mirrors how Wagner Houses' infestation began.</p>

	<figure class="chart">
		<figcaption class="chart-title">Total rodent complaints (Jan–May), by year</figcaption>

		<p class="section-label drop-label">Biggest drops</p>
		<div class="grid">
			{#each data.filter(d => d.type === 'drop').slice(0, 3) as panel}
				<svg viewBox="0 0 {width} {height}" width="100%">
					<!-- title -->
					<text x={margin.left} y="12" class="panel-title">
						{panel.address}
					</text>
					<text x={margin.left} y="28" class="panel-subtitle">
						{panel.borough}
					</text>

					<!-- bars -->
					{#each panel.yearTotals as d}
						<rect
							x={xScale(d.year)}
							y={yScaleDrop(d.count)}
							width={xScale.bandwidth()}
							height={height - margin.bottom - yScaleDrop(d.count)}
							fill={colorScale(panel.type)}
						/>
					{/each}

					<!-- year labels -->
					{#each panel.yearTotals as d}
						<text
							x={xScale(d.year) + xScale.bandwidth() / 2}
							y={height - margin.bottom + 15}
							text-anchor="middle"
							class="axis-label"
						>
							'{String(d.year).slice(2)}
						</text>
					{/each}

					<!-- y axis labels -->
					{#each yScaleDrop.ticks(3) as tick}
						<text
							x={margin.left - 5}
							y={yScaleDrop(tick) + 3}
							text-anchor="end"
							class="axis-label"
						>
							{tick}
						</text>
						<line
							x1={margin.left}
							x2={width - margin.right}
							y1={yScaleDrop(tick)}
							y2={yScaleDrop(tick)}
							stroke="#eee"
						/>
					{/each}
				</svg>
			{/each}
		</div>

		<p class="section-label spike-label">Biggest spikes</p>
		<div class="grid">
			{#each data.filter(d => d.type === 'spike').slice(0, 3) as panel}
				<svg viewBox="0 0 {width} {height}" width="100%">
					<!-- title -->
					<text x={margin.left} y="12" class="panel-title">
						{panel.address}
					</text>
					<text x={margin.left} y="28" class="panel-subtitle">
						{panel.borough}
					</text>

					<!-- bars -->
					{#each panel.yearTotals as d}
						<rect
							x={xScale(d.year)}
							y={yScaleSpike(d.count)}
							width={xScale.bandwidth()}
							height={height - margin.bottom - yScaleSpike(d.count)}
							fill={colorScale(panel.type)}
						/>
					{/each}

					<!-- year labels -->
					{#each panel.yearTotals as d}
						<text
							x={xScale(d.year) + xScale.bandwidth() / 2}
							y={height - margin.bottom + 15}
							text-anchor="middle"
							class="axis-label"
						>
							'{String(d.year).slice(2)}
						</text>
					{/each}

					<!-- y axis labels -->
					{#each yScaleSpike.ticks(3) as tick}
						<text
							x={margin.left - 5}
							y={yScaleSpike(tick) + 3}
							text-anchor="end"
							class="axis-label"
						>
							{tick}
						</text>
						<line
							x1={margin.left}
							x2={width - margin.right}
							y1={yScaleSpike(tick)}
							y2={yScaleSpike(tick)}
							stroke="#eee"
						/>
					{/each}
				</svg>
			{/each}
		</div>

		<figcaption class="chart-footer">Source: NYC 311 Service Requests (Open Data)</figcaption>
	</figure>
</article>

<style>
	.story-card {
		margin-bottom: 3rem;
	}

	.divider {
		border: none;
		border-top: 1px solid #222;
		margin: 0 0 2rem;
	}

	.headline {
		font-family: 'Source Serif 4', serif;
		font-size: 1.75rem;
		font-weight: 700;
		line-height: 1.2;
		margin: 0 0 0.5rem;
	}

	.byline {
		font-size: 0.875rem;
		color: #666;
		margin: 0 0 1.5rem;
	}

	p {
		font-size: 1.0625rem;
		line-height: 1.7;
		margin: 0 0 1rem;
	}

	.chart {
		margin: 2rem 0 0;
	}

	.chart-title {
		font-size: 0.875rem;
		font-weight: 600;
		margin-bottom: 0.75rem;
	}

	.grid {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 0;
	}

	.section-label {
		font-size: 1.125rem;
		font-weight: 700;
		margin: 0.5rem 0 0.25rem;
	}

	.drop-label {
		color: #4e79a7;
	}

	.spike-label {
		color: #e15759;
	}

	.panel-title {
		font-size: 15px;
		font-weight: 600;
		fill: #333;
	}

	.panel-subtitle {
		font-size: 13px;
		fill: #666;
	}

	.axis-label {
		font-size: 13px;
		fill: #999;
	}

	.chart-workspace {
		display: flex;
		align-items: center;
		justify-content: center;
		min-height: 320px;
		border: 1px solid #ddd;
		background: #fafafa;
	}

	.cat-label {
		font-size: 12px;
		font-weight: 600;
		fill: #333;
	}

	.count-label {
		font-size: 11px;
		fill: #666;
	}

	.chart-footer {
		margin-top: 0.75rem;
		font-size: 0.8125rem;
		color: #666;
	}
</style>
