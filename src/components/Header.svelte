<script>
    import {
      SkipToContent,
      Header,
      HeaderUtilities,
    } from "carbon-components-svelte";
    import { getContext } from "svelte";
  
    const ctx = getContext("Theme");
  
    $: if (ctx) {
      ctx.updateVar("--cds-productive-heading-06-font-size", "4rem");
    }
  </script>
  
  <Header platformName="Writing Diff" href="/">
    <div slot="skip-to-content">
      <SkipToContent />
    </div>
    <HeaderUtilities>
    </HeaderUtilities>
  </Header>