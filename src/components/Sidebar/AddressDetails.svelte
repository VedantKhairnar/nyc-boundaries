<script lang="ts">
  import SidebarHeader from './SidebarHeader.svelte';
  import { addressMarker, selectedAddress, mapStore } from '../../stores';
  import OverlapList from './OverlapList.svelte';
  import type { Feature } from 'geojson';
  import { resetZoom } from '../../helpers/helpers';

  let districtsIntersectingAddress: Feature[];
  let isLoading = false;

  async function queryAllDistrictsForCoordinates(lng: number, lat: number) {
    districtsIntersectingAddress = [];
    isLoading = true;
    //const intersectsUrl = `https://betanyc.carto.com/api/v2/sql/?q=SELECT * FROM all_bounds WHERE ST_Intersects(ST_SetSRID(ST_MakePoint(${lng}, ${lat}), 4326),the_geom)&api_key=2J6__p_IWwUmOHYMKuMYjw&format=geojson`;
    const intersectsUrl = `https://yhatmsxmjxmpgnnzdrzy.supabase.co/rest/v1/rpc/address_details?apikey=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InloYXRtc3htanhtcGdubnpkcnp5Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDM2OTA4OTQsImV4cCI6MjA1OTI2Njg5NH0.03AZcgwuHf2fAzIuCq8-O8UcSGVGfmvNdMYT6FH08b0&p_lng=${lng}&p_lat=${lat}`;
    const options = {
      headers: {
        'Accept': 'application/geo+json'
      }
    };
    await fetch(intersectsUrl, options)
      .then(res => res.json())
      .then(({ features }) => {
        isLoading = false;
        districtsIntersectingAddress = features;
      });
  }

  function handleBack() {
    selectedAddress.set(null);
    $addressMarker.remove();
    resetZoom($mapStore);
  }

  $: $selectedAddress &&
    queryAllDistrictsForCoordinates(
      $selectedAddress.coords[0],
      $selectedAddress.coords[1]
    );
</script>

<SidebarHeader
  title={$selectedAddress ? $selectedAddress.name : 'Loading&hellip;'}
  onBack={handleBack}
/>
<div class="py-2">
  <OverlapList districts={districtsIntersectingAddress} {isLoading} />
</div>
