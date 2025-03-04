<script lang="ts">
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { feature } from 'topojson-client';
  import { browser } from '$app/environment';
  import Tooltip from '../atoms/Tooltip.svelte';
  import provinces from '../../util/provinces.json';

  let width = 600;
  let height = 1000;
  let selectedProvince: { 
    name: string; 
    details: string[]; 
    id: string 
  } | null = null;
  let svg: any;
  
  onMount(async () => {
    if (browser) {
      const response = await fetch('/thailand-provinces.topojson');
      if (!response.ok) throw new Error('Failed to fetch TopoJSON');
      const topojsonData = await response.json();
      const thaiProvinces = feature(topojsonData, topojsonData.objects.province);

      const projection = d3.geoMercator()
        .scale(3000)
        .center([100.5018, 15.8700])
        .translate([width / 2 - 125, height / 2 - 210]);

      const path = d3.geoPath().projection(projection);

      svg = d3.select('#map')
        .append('svg')
        .attr('width', width)
        .attr('height', height);

      svg.selectAll('path')
        .data(thaiProvinces.features)
        .enter()
        .append('path')
        .attr('d', path)
        .attr('class', 'province')
        .attr('id', d => d.properties.ID_1)
        .attr('fill', d => {
          const province = provinces.find(p => p.id === d.properties.ID_1);
          return province ? (province.filled ? 'var(--text-primary)' : 'var(--text-secondary)') : 'var(--accent-opacity)';
        })
        .on('click', (event, d) => {
          const province = provinces.find(p => p.id === d.properties.ID_1);
          if (province) {
            selectedProvince = {
              name: province.name,
              details: province.details || ['No details available'],
              id: province.id
            };
          }
        });
    }
  });

  function closeModal(event: MouseEvent | KeyboardEvent) {
    if (event.type === 'click' || 
        (event as KeyboardEvent).key === 'Enter' || 
        (event as KeyboardEvent).key === ' ') {
      selectedProvince = null;
    }
  }
</script>

<style lang="scss">
  :root {
    --text-primary: #4e201c;
    --text-secondary: #744b40;
    --accent: #8b4d3f;
    --accent-opacity: #783b2e2a;
    --bg-color: #eae5db;
    --elevation-one: #e3dcd1;
    --elevation-two: #e3dcd175;
    --elevation-three: #92797340;
    --elevation-four: #d5c8bb;
    --elevation-five: #dfd8ccce;
    --elevation-six: #dbd4d09e;
  }

  span {
		color: var(--accent);
	}

	h2 {
		display: inline-block;
		margin-bottom: 1rem;
	}

  .container {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
  }

  .content {
    display: flex;
    flex-direction: row;
    gap: 20px;
  }

  .left-text {
    flex: 1;
    padding: 20px;
  }

  .modal {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: var(--bg-color);
    padding: 20px;
    border: 1px solid var(--elevation-one);
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
    z-index: 1000;
    max-height: 80vh;
    overflow-y: auto;
  }

  .modal-backdrop {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    z-index: 999;
  }

  .modal ul {
    list-style-type: disc;
    padding-left: 20px;
    margin: 10px 0;
  }

  .modal li {
    margin: 5px 0;
  }

  .title {
		display: flex;
		justify-content: center;
		margin-top: 0;

		@media (max-width: 868px) {
			justify-content: left;
		}
	}

  li {
    margin: 10px 0;
    font-size: 1.1re m;
  }
  
</style>

<div class="container">
  <div class="title">
		<Tooltip tip="Click on an province">
			<h2><span>competitive</span>:map</h2>
		</Tooltip>
  </div>
  
  <div class="content">
    <div class="left-text">
      <h2>Competition list</h2>
      <li>Llama 3 Hackathon Hackathon Certificate</li>
      <li>SuperAI Engineer Season 4</li>
      <li>Jump Thailand Hackathon 2024</li>
      <li>Hackathon Onsite : AI for Human Activity Recognition in Wellness Management</li>
      <li>Hackathon Onsite: Epidemiology Final</li>
      <li>Hackathon Onsite: Durian Hackathon 2025</li>
      <li>Gold Medal with project Disease detection in durian from aerial photographs : THAILAND NEW GEN INVENTORS AWARD 2023 </li>
      <li>Gold Medal with project Semiautomatic solar-powered water Surface garbage collecting machine : THAILAND NEW GEN INVENTORS AWARD 2024 </li>
    </div>
    
      <div id="map"></div>
  </div>
</div>

{#if selectedProvince}
  <div
    class="modal-backdrop"
    role="button"
    tabindex="0"
    on:click={closeModal}
    on:keydown={closeModal}
  ></div>

  <div class="modal">
    <h2>{selectedProvince.name}</h2>
    <ul>
      {#each selectedProvince.details as detail}
        <li>{detail}</li>
      {/each}
    </ul>
    <button on:click={closeModal}>ปิด</button>
  </div>
{/if}