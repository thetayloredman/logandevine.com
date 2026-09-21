+++
template = "homepage.html"

[[extra.buttons]]
url = "https://codestorm.net/"
src = "https://codestorm.net/buttons/sky.gif"
alt = "Sky"
+++

<style>
.homepage-hero {
    text-align: center;
    padding: 2rem 0;
}

.homepage-hero-title {
    font-size: 3rem;
    margin-bottom: 1rem;
}

.homepage-hero-subtitle {
    font-size: 1.25rem;
    margin-bottom: 1rem;
}

</style>

<div class="homepage-hero">
    <h1 class="homepage-hero-title">Hi, I'm LogN.</h1>
    <p class="homepage-hero-subtitle">
        Open-source developer and hobbyist, working on
        <a href="/projects">programming languages</a>,
        <a href="https://matrix.org" target="_blank">Matrix</a>,
        and self-hosted infrastructure.
    </p>
</div>

Hi! I'm Logan, but you probably know me online as LogN. I advocate for an open Internet, through
work on open-source software, selfhosting, and the [Matrix](https://matrix.org) open standard.
Occasionally, I write about that stuff. In my free time, I enjoy working on compilers and toying
with programming languages, like [Zirco](https://zirco.dev/). At my day job, I work as a general
manager at a small independent cinema!

[Come say hi!](/contact)

## Buttons!

<a href="https://logandevine.com/" target="_blank" class="button88x31">
    <img src="/images/88x31.gif" alt="LogN" width="88" height="31" />
</a>

Other cool people worth visiting:

{{ button(
    url="https://codestorm.net/",
    src="https://codestorm.net/buttons/sky.gif",
    alt="Sky"
) }}
{{ button(
    url="https://squarebowl.club/",
    src="https://squarebowl.club/images/88x31/plate.gif",
    alt="Plate"
) }}
