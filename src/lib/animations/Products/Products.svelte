<script lang="ts" context="module">
    export const elId = writable(0);

    export function getElSelector(el: string) {
        return `#${el}-${get(elId)}`;
    }

    /* Products infos */
    const products = ['auth', 'databases', 'storage', 'functions', 'realtime', 'post'] as const;
    type Product = (typeof products)[number];

    type ProductInfo = {
        icon: {
            active: string;
            inactive: string;
        };
        title: string;
        subtitle: string;
        description: string;
        features: string[];
        shot?: string;
    };
    export const infos: { [K in Product]?: ProductInfo } = {
        auth: {
            icon: {
                active: './images/icons/illustrated/dark/auth.png',
                inactive: './images/icons/illustrated/dark/auth-transparent.png'
            },
            title: 'Auth',
            subtitle: 'Secure login for all users',
            description:
                'Authenticate users securely with multiple login methods like Email/Password, SMS, OAuth, Annoymous, Magic URLs and more.',
            features: [
                '30+ login methods',
                'Support for teams, roles and user labels',
                'Rate-limits and advanced user protection',
                'Custom SMTP and email templates'
            ],
            shot: './images/products/auth.png'
        },
        databases: {
            icon: {
                active: './images/icons/illustrated/dark/databases.png',
                inactive: './images/icons/illustrated/dark/databases-transparent.png'
            },
            title: 'Databases',
            subtitle: 'Store, query and manage data',
            description: 'Scalable and robust database backed by your favorite technologies.',
            features: [
                'Never paused',
                'Fast in-memory caching',
                'Advanced permission models',
                'Custom data validation',
                'Relationships support'
            ],
            shot: './images/products/databases.png'
        },
        functions: {
            icon: {
                active: './images/icons/illustrated/dark/functions.png',
                inactive: './images/icons/illustrated/dark/functions-transparent.png'
            },
            title: 'Functions',
            subtitle: 'Customize and extend your backend',
            description: 'Deploy and scale serverless functions in secure, isolated runtimes.',
            features: [
                'Automatic deployment from GitHub',
                'Trigger using GitHub, CLI, Event Listeners or HTTP requests',
                'Support for 30+ runtimes in 13 languages',
                'Custom domain support'
            ],
            shot: './images/products/functions.png'
        },
        storage: {
            icon: {
                active: './images/icons/illustrated/dark/storage.png',
                inactive: './images/icons/illustrated/dark/storage-transparent.png'
            },
            title: 'Storage',
            subtitle: 'Upload and manage files',
            description:
                'Securely store files with advanced compression, encryption and image transformations.',
            features: [
                'File encryption at rest and transit',
                'Built-in image transformation capabilities',
                'Advanced compression with WebP/Brotli support'
            ],
            shot: './images/products/storage.png'
        },
        realtime: {
            icon: {
                active: './images/icons/illustrated/dark/realtime.png',
                inactive: './images/icons/illustrated/dark/realtime-transparent.png'
            },
            title: 'Realtime',
            subtitle: 'Realtime events for every service',
            description: 'Subscribe and react to any Appwrite event using the Realtime API.',
            features: [
                'Unlimited subscriptions',
                'Built-in permission management',
                'Support for DBs, Auth, Storage & Functions'
            ],
            shot: './images/products/realtime.png'
        }
    };
</script>

<script lang="ts">
    import { toScale, type Scale } from '$lib/utils/toScale';

    import { fly, slide } from 'svelte/transition';
    import { scroll } from '..';
    import ScrollIndicator from '../scroll-indicator.svelte';
    import { Auth, authController } from './auth';

    import AnimatedBox from './AnimatedBox.svelte';
    import { tick } from 'svelte';
    import CodeWindow from '../CodeWindow/CodeWindow.svelte';
    import { Databases, databasesController } from './databases';
    import { objectKeys } from '$lib/utils/object';
    import { get, writable } from 'svelte/store';
    import { Storage, storageController } from './storage';
    import { Functions, functionsController } from './functions';
    import { Realtime, realtimeController } from './realtime';
    import { postController } from './post';
    import Post from './post/post.svelte';
    import { anyify } from '$lib/utils/anyify';

    /* Basic Animation setup */
    let scrollInfo = {
        percentage: 0,
        traversed: 0,
        remaning: Infinity
    };

    const animScale: Scale = [0.075, 1];
    const productsScales = products.map((_, idx) => {
        const diff = animScale[1] - animScale[0];
        const step = diff / products.length;
        return [animScale[0] + step * idx, animScale[0] + step * (idx + 1)] as Scale;
    });

    $: active = (function getActiveInfo() {
        let activeIdx = productsScales.findIndex(([min, max]) => {
            return scrollInfo.percentage >= min && scrollInfo.percentage < max;
        });

        const product = products[activeIdx] as Product | undefined;
        const scale = productsScales[activeIdx] as Scale | undefined;
        const percent = scale ? toScale(scrollInfo.percentage, scale, [0, 1]) : 0;

        return {
            product,
            scale,
            percent
        };
    })();

    let lastActive: Product | undefined = undefined;

    const controllers = {
        auth: authController,
        databases: databasesController,
        storage: storageController,
        functions: functionsController,
        realtime: realtimeController,
        post: postController
    };

    $: (async () => {
        const fixedLast = lastActive;
        lastActive = active.product;

        objectKeys(controllers).forEach(async (key) => {
            const controller = controllers[key];
            if (active.product === key && fixedLast !== key) {
                if (!(fixedLast === 'realtime' && key === 'post')) {
                    elId.update((n) => n + 1);
                }
                await tick();
                controller.execute();
            }
        });
    })();
</script>

<div
    id="products"
    use:scroll
    on:aw-scroll={({ detail }) => {
        scrollInfo = detail;
    }}
>
    <div class="sticky-wrapper">
        <!-- <div class="debug">
			<pre>{scrollInfo.percentage}</pre>
		</div> -->
        {#if scrollInfo.percentage < 0.075}
            <div
                class="main-text"
                out:fly={{ duration: 250, y: -300 }}
                in:fly={{ duration: 250, delay: 250, y: -300 }}
            >
                {#if scrollInfo.percentage > -0.1}
                    <span class="aw-badges aw-eyebrow" transition:slide={{ axis: 'x' }}
                        >Products_</span
                    >

                    <h2
                        class="aw-display aw-u-color-text-primary"
                        transition:fly={{ y: 16, delay: 250 }}
                    >
                        Your backend, minus the hassle
                    </h2>
                    <p
                        class="aw-description aw-u-max-width-700 u-margin-inline-auto"
                        transition:fly={{
                            y: 16,
                            delay: 400
                        }}
                    >
                        Build secure and scalable applications with less code. Add authentication,
                        databases, storage, and more using Appwrite's development platform.
                    </p>
                {/if}
            </div>
        {:else}
            <div
                class="products"
                out:fly={{ duration: 250, y: 300 }}
                in:fly={{ duration: 500, delay: 250, y: 300 }}
                data-active={scrollInfo.percentage > 0.075 ? '' : undefined}
            >
                <div class="text" id="pd-{$elId}">
                    <ScrollIndicator
                        percentage={toScale(scrollInfo.percentage, animScale, [0, 1])}
                    />
                    <ul class="descriptions">
                        {#each products as product}
                            {@const copy = infos[product]}
                            {@const isActive = active.product === product}

                            {#if copy}
                                <li data-active={isActive ? '' : undefined}>
                                    <h3>
                                        <img
                                            src={isActive ? copy.icon.active : copy.icon.inactive}
                                            alt=""
                                            width="32"
                                            height="32"
                                        />
                                        <span class="aw-label">{copy.title}</span>
                                    </h3>
                                    {#if isActive}
                                        <div transition:slide>
                                            <h4 class="aw-title">{copy.subtitle}</h4>
                                            <p>
                                                {copy.description}
                                            </p>
                                            <ul class="features">
                                                {#each copy.features as feature}
                                                    <li>{feature}</li>
                                                {/each}
                                            </ul>
                                        </div>
                                    {/if}
                                </li>
                            {/if}
                        {/each}
                    </ul>
                </div>

                <div class="animated">
                    <div class="box-wrapper" id="box-{$elId}">
                        <AnimatedBox>
                            <div class="top" slot="top">
                                <p class="title">
                                    {#if active.product === 'auth'}
                                        Users
                                    {:else if active.product === 'databases'}
                                        Tasks
                                    {:else if active.product === 'storage'}
                                        Files
                                    {:else if active.product === 'functions'}
                                        <!-- oblivion -->
                                    {:else if active.product === 'realtime'}
                                        Realtime
                                    {/if}
                                </p>
                            </div>

                            {#if active.product === 'auth'}
                                <Auth.Box />
                            {:else if active.product === 'databases'}
                                <Databases.Box />
                            {:else if active.product === 'storage'}
                                <Storage.Box />
                            {/if}
                        </AnimatedBox>
                    </div>

                    <div class="code-window" id="code-{$elId}">
                        <CodeWindow>
                            {#if active.product === 'auth'}
                                <Auth.Code />
                            {:else if active.product === 'databases'}
                                <Databases.Code />
                            {:else if active.product === 'storage'}
                                <Storage.Code />
                            {:else if active.product === 'functions'}
                                <Functions.Code />
                            {/if}
                        </CodeWindow>
                    </div>

                    {#if active.product === 'auth'}
                        <div class="controls" id="controls-{$elId}">
                            <Auth.Controls />
                        </div>
                    {/if}
                </div>

                <div class="phone" id="phone-{$elId}">
                    <div class="inner">
                        {#if active.product === 'auth'}
                            <Auth.Phone />
                        {:else if active.product === 'databases'}
                            <Databases.Phone />
                        {:else if active.product === 'storage'}
                            <Storage.Phone />
                        {:else if active.product === 'functions'}
                            <Functions.Phone />
                        {:else if !['auth', 'databases', 'storage', 'functions'].includes(anyify(active.product))}
                            <Realtime.Phone />
                        {/if}
                    </div>
                </div>

                {#if !['auth', 'databases', 'storage', 'functions', 'realtime'].includes(anyify(active.product))}
                    <Post />
                {/if}
            </div>
        {/if}
    </div>
</div>

<style lang="scss">
    #products {
        min-height: 500vh;
        height: 5000px;
        position: relative;

        --debug-bg: transparent;

        display: none;
    }

    @media (min-width: 1400px) {
        #products {
            display: block;
        }
    }

    .sticky-wrapper {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: space-around;
        gap: 1rem;

        position: sticky;

        top: 0;
        // min-height: 58rem;
        overflow: hidden;

        padding-inline: 1.25rem;

        width: 100%;
        height: 100vh;

        > .main-text {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);

            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 15rem;
            text-align: center;

            h2 {
                white-space: nowrap;
                margin-top: 1.5rem;
            }

            p {
                margin-top: 1.25rem;
                max-width: 48.875rem;
            }

            @media (min-width: 1024px) {
                h2 {
                    max-width: 61.375rem;
                }
            }
        }
    }

    .products:not([data-active]) {
        opacity: 0;
    }

    .products[data-active] {
        opacity: 1;
    }

    .products {
        background: var(--debug-bg, hsla(250, 50%, 50%, 0.25));

        display: flex;
        justify-content: space-between;
        width: 100%;
        max-width: 77.75rem;
        position: relative;

        transition: 200ms ease;

        .text {
            background: var(--debug-bg, hsla(200, 50%, 50%, 0.25));
            display: flex;
            flex-grow: 1;
            max-width: 25rem;
            position: relative;

            .descriptions {
                margin-inline-start: 2rem;
                text-align: left;
                position: absolute;
                width: 100%;

                > li {
                    &:not(:first-child) {
                        padding-block-start: 1.5rem;
                    }

                    transition: 100ms ease;

                    &[data-active] {
                        h3 {
                            color: hsl(var(--aw-color-primary));
                            margin-block-end: 0.75rem;
                        }
                    }
                }

                h3 {
                    display: flex;
                    align-items: center;
                    gap: 0.75rem;

                    .aw-label {
                        margin-block-start: 0.25rem;
                    }
                }

                h4 {
                    color: hsl(var(--aw-color-primary));
                }

                p {
                    margin-block-start: 1rem;
                }

                .features {
                    display: flex;
                    flex-direction: column;
                    gap: 0.75rem;
                    margin-block-start: 2rem;

                    li {
                        --marker-size: 1.25rem;
                        --margin-left: calc(var(--marker-size) + 0.75rem);
                        position: relative;
                        margin-inline-start: var(--margin-left);

                        &::before {
                            content: '';
                            position: absolute;

                            left: calc(var(--margin-left) * -1);
                            top: 50%;
                            width: var(--marker-size);
                            height: var(--marker-size);

                            transform: translateY(-50%);

                            background: url('/images/icons/colored/check.svg') no-repeat;
                        }
                    }
                }
            }
        }
    }

    .animated {
        background: var(--debug-bg, hsl(100, 50%, 50%, 0.25));
        width: min(42rem, 50vw);
        height: min(38.75rem, 90vh);

        position: relative;
    }

    .phone {
        @include border-gradient;
        --m-border-size: 1px;
        --m-border-radius: 2.5rem;
        --m-border-gradient-after: linear-gradient(
            180deg,
            rgba(255, 255, 255, 0.12) 0%,
            rgba(255, 255, 255, 0) 125.11%
        );

        background: rgba(255, 255, 255, 0.08);
        backdrop-filter: blur(8px);
        padding: 0.5rem;

        width: 275px;
        height: 550px;
        flex-shrink: 0;

        position: absolute;
        top: 0;
        left: calc(50%);
        z-index: 10;
        opacity: 1;

        .inner {
            background-color: white;
            border-radius: 2rem;
            width: 100%;
            height: 100%;

            position: relative;

            &::after {
                content: '';
                position: absolute;
                left: 50%;
                transform: translateX(-50%);
                bottom: 0.5rem;

                border-radius: 100rem;
                background: var(--label-color-light-primary, #000);

                width: 6.25rem;
                height: 0.25rem;
            }
        }
    }

    .box-wrapper {
        position: absolute;
        top: 0;
        z-index: 0;
        opacity: 0;

        width: 25rem;

        transform: translateX(16.5rem) translateY(2rem);
    }

    .box-wrapper :global(.pseudo-table) {
        :global(.header),
        :global(.row) {
            display: grid;
            grid-template-columns: 10rem 1fr;
            justify-content: space-between;
            align-items: center;
            gap: 1.5rem 3rem;
        }

        :global(.header) {
            border-bottom: 1px solid hsl(var(--aw-color-greyscale-700));
            color: var(--greyscale-400, #adadb1);

            text-transform: uppercase;
            padding: 1rem;
        }

        :global(.row) {
            padding-block: 0.5rem;
            padding-inline: 1rem;

            color: var(--greyscale-400, #adadb1);
            font-family: Inter;
            font-size: 0.875rem;
            font-style: normal;
            font-weight: 400;
            line-height: 1.25rem; /* 142.857% */
        }

        :global(.avatar) {
            background-color: hsl(var(--aw-color-greyscale-700));
            border-color: hsl(var(--aw-color-greyscale-700));
        }

        :global(.truncated) {
            display: block;
            text-overflow: ellipsis;
            overflow: hidden;
            white-space: nowrap;
        }
    }

    .code-window {
        position: absolute;
        z-index: 20;
        top: 0;
        left: 0;
        opacity: 0;
    }

    .controls {
        @include border-gradient;
        --m-border-radius: 1rem;
        --m-border-gradient-before: linear-gradient(
            180deg,
            rgba(255, 255, 255, 0.12) 0%,
            rgba(255, 255, 255, 0) 125.11%
        );

        position: absolute;
        top: 0;
        left: 0;
        z-index: 10;
        opacity: 0;

        background: rgba(255, 255, 255, 0.08);
        box-shadow: 0px 0px 0px 0px rgba(0, 0, 0, 0.06), -2px 4px 9px 0px rgba(0, 0, 0, 0.06),
            -8px 15px 17px 0px rgba(0, 0, 0, 0.05), -19px 34px 23px 0px rgba(0, 0, 0, 0.03),
            -33px 60px 27px 0px rgba(0, 0, 0, 0.01), -52px 94px 30px 0px rgba(0, 0, 0, 0);
        backdrop-filter: blur(20px);
    }
</style>
