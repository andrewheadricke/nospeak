<script lang="ts">
    import { getIdenticonDataUri } from '$lib/core/identicon';
    import { buildBlossomGetAuthHeader } from '$lib/core/BlossomAuth';

    let { src, npub, size = 'md', class: className = '', style = '' } = $props<{ 
        src?: string, 
        npub: string, 
        size?: 'sm' | 'md' | 'lg' | 'xl' | '2xl',
        class?: string,
        style?: string
    }>();

    const identiconSrc = $derived(getIdenticonDataUri(npub));
    
    let imgError = $state(false);
    
    // If src provided and no error, use it. Otherwise fallback to identicon.
    const finalSrc = $derived(!imgError && src ? src : identiconSrc);

    // Reset error state when src changes
    $effect(() => {
        src;
        imgError = false;
    });
    
    // Size classes
    const sizeClasses = {
        sm: 'w-8 h-8',
        md: 'w-10 h-10',
        lg: 'w-16 h-16',
        xl: 'w-24 h-24',
        '2xl': 'w-32 h-32'
    };

    let authTried = false;
    function handleImgError(e: error, npub: string) {
        console.log("image load error", e, npub);
        imgError = true
        if (!authTried) {
          tryAuthDownload()
        }
    }

    async function tryAuthDownload() {
      authTried = true;
      let destUrl = URL.parse(src)
      let sha256 = destUrl.pathname.substring(1)
      if (sha256.length != 64) {
        console.log("invalid blossom hash")
        return
      }
      const token = await buildBlossomGetAuthHeader({sha256: sha256})
      const response = await fetch(src, {
        method: 'GET',
        headers: {
          'Authorization': token
        }
      });
      const imageBlob = await response.blob();
      const objectURL = URL.createObjectURL(imageBlob);
      src = objectURL;
      imgError = false;
    }
</script>

<div class={`${sizeClasses[size as keyof typeof sizeClasses]} ${className} rounded-full ring-2 ring-white/50 dark:ring-white/10 overflow-hidden flex-shrink-0 bg-gray-200 dark:bg-slate-700 shadow-sm`} style={style}>
    <img 
        src={finalSrc} 
        alt="Avatar" 
        class="w-full h-full object-cover"
        onerror={(e) => handleImgError(e, npub)}
    />
</div>
