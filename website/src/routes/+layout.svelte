<script lang="ts">
	import '../app.css';

	import * as Sidebar from '$lib/components/ui/sidebar/index.js';
	import { Toaster } from '$lib/components/ui/sonner';

	import AppSidebar from '$lib/components/self/AppSidebar.svelte';

	import { USER_DATA } from '$lib/stores/user-data';
	import { onMount } from 'svelte';
	import { invalidateAll } from '$app/navigation';
	import { ModeWatcher } from 'mode-watcher';
	import { page } from '$app/state';
	import { websocketController } from '$lib/stores/websocket';
	import { PUBLIC_DEMO_MODE } from '$env/static/public';

	let { data, children } = $props<{
		data: { userSession?: any };
		children: any;
	}>();

	USER_DATA.set(data?.userSession ?? null);

	$effect(() => {
		USER_DATA.set(data?.userSession ?? null);
	});

	// Check if demo mode is enabled
	const isDemoMode = PUBLIC_DEMO_MODE === 'true';

	onMount(() => {
		websocketController.connect();

		console.log(
			`%c                                       .--                    
                                      .=--:                   
                                   :=*#*:                     
                               .=******+#*.                   
                            .+*****+*#*+**#*                  
                          :**++**####*###*++#-                
                        =***+*####******###*+#*               
                      =***++#####***+++***%#*+*%:             
                    =*++*###+=++++====****##%#**#=            
                 .+**+=*##=*###+####*#+++*###%#**#=           
               :#*=**####=*#+-*##=-*##+**#####%##*%=          
     .      :+**++*###***++=*#++=*###**######%%%####:.--:     
    .---=******+*###****=***=-**+##*#+*###%%%***##%%#=--:     
     :-:  =#++**##***+++=******#*=##**#%%%##*#%*:             
           .**++*##***++**+**#*####+*%%#**#%+.                
             +***+##*=**=++******##%%*####:                   
              -#+++###***+*######%####%+                      
               .#*++*##**#####%%#**##=                        
                 *#*+*######%%#*###=                          
                  +#**#%%%%##**##-                            
                   =#***#*###%+.                              
                    -%#####*:                                 
                    .=%#*:                                    
                 .=--=.                                       
                   ::`,
			'color: #4962ee; font-family: monospace; font-size: 12px; font-weight: bold; text-shadow: 2px 2px rgba(0,0,0,0.2);'
		);
		console.log(
			'%c Welcome to Rugplay! DO NOT FUCKING PASTE ANYTHING IN THE CONSOLE UNLESS YOU KNOW WHAT YOU ARE DOING.',
			'color: #4962ee; font-family: monospace; font-size: 12px; font-weight: bold; text-shadow: 2px 2px rgba(0,0,0,0.2);'
		);		console.log(
			'%c A product by Outpoot.com',
			'color: #4962ee; font-family: monospace; font-size: 12px; font-weight: bold; text-shadow: 2px 2px rgba(0,0,0,0.2);'
		);

		if (isDemoMode) {
			console.log(
				'%c 🎭 DEMO MODE ACTIVE - Google OAuth not required. Use "Continue as Demo User" to sign in.',
				'color: #f59e0b; font-family: monospace; font-size: 12px; font-weight: bold; background: #fef3c7; padding: 4px;'
			);
		}

		const url = new URL(window.location.href);
		if (url.searchParams.has('signIn')) {
			url.searchParams.delete('signIn');
			window.history.replaceState({}, '', url);
			invalidateAll();
		}

		return () => {
			websocketController.disconnect();
		};
	});

	function getPageTitle(routeId: string | null): string {
		if (!routeId) return 'Rugplay';

		const titleMap: Record<string, string> = {
			'/': 'Home',
			'/market': 'Market',
			'/portfolio': 'Portfolio',
			'/leaderboard': 'Leaderboard',
			'/coin/create': 'Create Coin',
			'/settings': 'Settings',
			'/admin': 'Admin',
			'/admin/promo': 'Promo Codes',
			'/transactions': 'Transactions',
			'/hopium': 'Hopium',
			'/gambling': 'Gambling',
			'/live': 'Live Trades',
			'/treemap': 'Treemap'
		};

		// Handle dynamic routes
		if (routeId.startsWith('/coin/[coinSymbol]')) {
			return 'Coin Details';
		}
		if (routeId.startsWith('/user/[username]')) {
			return 'User Profile';
		}
		if (routeId.startsWith('/hopium/[id]')) {
			return 'Prediction Question';
		}

		return titleMap[routeId] || 'Rugplay';
	}
</script>

<ModeWatcher />
<Toaster richColors={true} />

<Sidebar.Provider>
	<AppSidebar />

	<Sidebar.Inset class="sidebar-container">
		{#if isDemoMode}
			<div class="bg-yellow-100 border-b border-yellow-200 px-4 py-2 text-center">
				<p class="text-yellow-800 text-sm">
					🎭 <strong>Demo Mode</strong> - You're using a demo version without Google OAuth setup
				</p>
			</div>
		{/if}

		<header
			class="group-has-data-[collapsible=icon]/sidebar-wrapper:h-12 flex h-12 shrink-0 items-center gap-2 border-b transition-[width,height] ease-linear"
		>
			<div class="flex w-full items-center gap-4 px-4 lg:px-6">
				<Sidebar.Trigger class="-ml-1" />

				<h1 class="mr-6 text-base font-medium">
					{getPageTitle(page.route.id)}
				</h1>
			</div>
		</header>

		<div class="main-content-area">
			<div class="@container/main flex flex-col gap-2">
				<div class="flex flex-col gap-4 md:gap-6">
					<div class="px-4 md:py-4 lg:px-6">
						{@render children()}
					</div>
				</div>
			</div>
		</div>
	</Sidebar.Inset>
</Sidebar.Provider>
