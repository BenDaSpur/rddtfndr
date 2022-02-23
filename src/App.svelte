<script>
    import browser from "webextension-polyfill";
    import {
        Row,
        Col,
        Container,
        Button,
        TabContent,
        TabPane,
    } from "sveltestrap";
    import { onMount } from "svelte";

    function goToPost(newUrl) {
        chrome.tabs.update({ url: `https://reddit.com${newUrl}` });
    }

    function goToSubreddit(theSub) {
        chrome.tabs.update({ url: `https://reddit.com/r/${theSub}` });
    }

    function reload() {
        window.location.reload();
    }

    const key = "background";
    browser.storage.local.get(key).then((data) => {
        // document.body.style = `background: url(${data[key]})`;
    });

    let postcount = 0;

    onMount(() => {
        chrome.tabs.query({ active: true, lastFocusedWindow: true }, (tabs) => {
            let url = tabs[0].url;
            currentUrl = url;
            console.log(url);
            localStorage.setItem("theUrl", url);
        });
    });

    const fetchImage = (async () => {
        // console.log(currentUrl);
        const response = await fetch(
            `https://www.reddit.com/submit.json?url=${encodeURIComponent(
                localStorage.getItem("theUrl")
            )}`
        );

        return await response.json();
    })();
</script>

<svelte:head>
    <link
        rel="stylesheet"
        href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.5.0/font/bootstrap-icons.css"
    />
</svelte:head>

<main>
    <Container class="the-container">
        <Button on:click={() => reload()}>Reload</Button>
        <Row>
            {#await fetchImage}
                <p>...waiting</p>
            {:then data}
                <!-- {data} -->
                {#if data && data.data && data.data.children && data.data.children.length > 0}
                    <!-- {currentUrl} -->
                    <!-- {JSON.stringify(data)} -->
                    <Col>
                        <TabContent vertical pills>
                            {#each data.data.children as post, count}
                                <TabPane
                                    tabId={post.data.subreddit + count}
                                    tab={post.data.subreddit}
                                    active={count == 0 ? "true" : "false"}
                                >
                                    <Col
                                        xs={6}
                                        class="position-fixed top-0 end-0"
                                    >
                                        <!-- <h2>{post.data.title}</h2> -->
                                        <Row>
                                            <Col>
                                                <a
                                                    href="#"
                                                    on:click={() =>
                                                        goToSubreddit(
                                                            post.data.subreddit
                                                        )}
                                                >
                                                    /r/{post.data.subreddit}
                                                </a>
                                            </Col>
                                        </Row>
                                        <Row>
                                            <Col>
                                                <a
                                                    href="#"
                                                    on:click={() =>
                                                        goToPost(
                                                            post.data.permalink
                                                        )}
                                                >
                                                    {post.data.title}
                                                </a>
                                            </Col>
                                        </Row>
                                    </Col>
                                </TabPane>
                            {/each}
                        </TabContent>
                    </Col>
                {:else}
                    No posts found
                {/if}
            {:catch error}
                {error}
                <p>An error occurred!</p>
            {/await}
        </Row>
    </Container>
</main>

<style>
    :global(.the-container) {
        /* max-width: 800px; */
        min-width: 500px;
        overflow: auto;
    }
</style>
