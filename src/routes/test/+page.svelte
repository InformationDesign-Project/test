<script lang="ts">
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
  
    interface UnitItem {
      address: string;
      moniker: string;
      website: string;
      token: number;
      commission: number;
      votingPower: number;
      rank: number;
      jailed: boolean;
    }
  
    interface DataItem {
      bondedToken: number;
      units: UnitItem[];
    }
  
    let data: DataItem[] = [];
    
    onMount(async () => {
      try {
        const response = await fetch('/Validators/validators_akash.json');
        if (!response.ok) {
          console.error('서버 응답 오류:', response.status);
          return;
        }
        const jsonData = await response.json();
        console.log('로딩된 데이터:', jsonData);
        // 데이터를 처음 50개 항목만 사용하도록 수정
        data = jsonData.data.slice(0, 50);
        drawTreemap();
      } catch (error) {
        console.error('데이터 로딩 중 오류 발생:', error);
      }
    });
  
    function drawTreemap() {
      if (data.length === 0) return;
  
      const flattenedData = data.flatMap((d) => d.units);
  
      const root = d3
        .hierarchy({ children: flattenedData } as any)
        .sum((d) => d.votingPower)
        .sort((a, b) => (b.value ?? 0) - (a.value ?? 0));
  
      d3.treemap()
        .size([800, 600])
        .padding(1)
        (root);
  
      const svg = d3.select('#treemap');
      const cell = svg.selectAll('g')
                      .data(root.leaves())
                      .enter().append('g')
                      .attr('transform', (d: any) => `translate(${d.x0},${d.y0})`);
  
      cell.append('rect')
         .attr('width', (d: any) => d.x1 - d.x0)
         .attr('height', (d: any) => d.y1 - d.y0)
         .attr('fill', 'steelblue');
  
      cell.append('text')
         .attr('x', 5)
         .attr('y', 20)
         .text((d: any) => d.data.moniker);
    }
  </script>
  
  <svg id="treemap" width="800" height="600"></svg>
  