---
layout: home
title: Infinibreak
permalink: /infinibreak/
short_desc: An action-packed infinite brick-breaking game
carousels:
- images:
    - image: /assets/infinibreak/screenshot1.png
    - image: /assets/infinibreak/screenshot2.png
    - image: /assets/infinibreak/screenshot3.png

---
<style>
    #game-header {
        display: grid;
        grid-template-columns: auto auto 225px;
        width: 100%;
        gap: 0px;
    }

    #game-content {
        grid-area: 1 / 1 / 2 / 3;
    }

    #game-screenshot {
        grid-area: 1 / 3 / 3 / 4;
    }

    .app-store-icon {
        width: 207px;
    }
    .app-store-icon img {
        padding:8px;
        width: 191px;
    }

</style>

<div id="game-header">
    <div id="game-content">
        <h1>Infinibreak</h1>
        <p>How long can you go?
            <ul>
                <li>The bricks keep on coming, but destroying them makes you more powerful.</li>
                <li>Dodge incoming attacks while trying to keep as many balls in going at once</li>
                <li>Reach for high scores as the blocks keep coming</li>
            </ul>
        </p>
    </div>

    <div class="app-store-icon">
        <a href="https://apps.apple.com/ca/app/infinibreak/id1553616537?itsct=apps_box_badge&amp;itscg=30200">
            <img src="/assets/apple_app_store.svg" alt="Download on the App Store">
        </a>
    </div>

    <div class="app-store-icon">
        <a href='https://play.google.com/store/apps/details?id=io.itch.michaelreldred.breakoutclone&pcampaignid=pcampaignidMKT-Other-global-all-co-prtnr-py-PartBadge-Mar2515-1'>
            <img alt='Get it on Google Play' src='/assets/google_play_store.svg' />
        </a>
    </div>

    <div id="game-screenshot">
        {% include carousel.html height="400" unit="px" duration="5" number="1" %}
    </div>
</div>

<div id="game-footer">
    <a href="/infinibreak/privacy/">Privacy Policy</a>
</div>