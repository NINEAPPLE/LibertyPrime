<script setup>
const Version = 'v1.0.0';


import { ref } from 'vue';

const helperClass = ref('');
let enterTimer = null;
let leaveTimer = null;

const handleMouseEnter = () => {
    clearTimeout(leaveTimer);
    helperClass.value = 'hover-enter';
    enterTimer = setTimeout(() => {
        helperClass.value = 'hover-enter-end';
    }, 80);
};

const handleMouseLeave = () => {
    clearTimeout(enterTimer);
    helperClass.value = 'hover-leave';
    leaveTimer = setTimeout(() => {
        helperClass.value = '';
    }, 150);
};

const versionEasterEggClass = ref('');
let versionEasterEggCount = 0;
const versionEasterEgg = () => {
    versionEasterEggCount ++;
    console.log(`${versionEasterEggCount}!`);
    if (versionEasterEggCount >= 10) {
        versionEasterEggClass.value = 'versionEasterEgg';
    };
};
</script>

<template>
    <h2>
        <span class="logo-wrapper" @mouseenter="handleMouseEnter" @mouseleave="handleMouseLeave">
            <span class="logo">LibertyPrime</span> <span :class="['titleHelper', helperClass]">Skin</span> License
        </span>
    </h2>
    <div class="github-wrapper">
        <a href="//github.com/NINEAPPLE/LibertyPrime" target="_blank" class="github"><span class="fa-brands fa-github" /> GitHub Repository</a>
    </div>
    <div class="version-wrapper"><span :class="['version', versionEasterEggClass]" @click="versionEasterEgg">{{ versionEasterEggClass ? '와! 샌즈! (놀랍게도이스터에그임)' : Version }}</span></div>

    <h3>원본 소스</h3>
    <ul>
        <li><a href="//github.com/wjdgustn/thetree-skin-liberty">thetree Liberty</a></li>
        <li><a href="//github.com/namu-theseed/theseed-skin-liberty">theseed Liberty</a></li>
        <li><a href="//github.com/librewiki/liberty-skin">Liberty Skin</a></li>
    </ul>
</template>

<style scoped>
.logo-wrapper {
    cursor: default;
}

.logo {
    color: transparent;
    background: text linear-gradient(to right in oklab, #d9bd43, #c97e28);
    font-weight: 900;
    border-radius: 5px;
    transition: .3s cubic-bezier(.1, 0, 0, 1);
}

.logo-wrapper:hover .logo {
    background: linear-gradient(to right in oklab, #d9bd43, #c97e28);
    padding: 0 15px;
    color: #fff;
}

.titleHelper {
    display: inline-block;
    transition: .3s cubic-bezier(.1, 0, 0, 1);
}

.titleHelper.hover-enter {
    margin-right: -1rem;
}

.titleHelper.hover-enter-end {
    margin-right: 0;
}

.titleHelper.hover-leave {
    margin: 0 -.5rem;
}

.github-wrapper {
    margin-top: -20px;
}

.github {
    font-size: 18px;
    font-weight: 600;
    color: transparent;
    background: text linear-gradient(135deg in oklab, #4188f1, #5038ba);
    --license-github-color: #000;
}

.theseed-dark-mode .github {
    --license-github-color: #fff;
}

.github:hover {
    color: var(--license-github-color);
    background: none;
}

.github:active {
    color: transparent;
    background: text linear-gradient(135deg in oklab, #4188f1, #5038ba);
}

.github:hover, .github:focus {
    text-decoration: none;
}

.github .fa-github {
    color: var(--license-github-color);
    transition: .3s transform cubic-bezier(.1, 0, 0, 1);
}

.github:hover:not(:active) .fa-github {
    transform: rotate(30deg) scale(1.2);
    color: transparent;
    background: text linear-gradient(135deg in oklab, #4188f1, #5038ba);
}

.version {
    font-size: 18px;
    font-weight: 700;
    transition: .3s font-size cubic-bezier(.1, 0, 0, 1);
    cursor: default;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    display: inline-block;
}

.version:hover:not(:active, .versionEasterEgg) {
    font-size: 22px;
    animation: gradient 5s linear infinite;
}

.version:hover:not(:active),
.version.versionEasterEgg {
    background: linear-gradient(-45deg, #31c4b6, #314ac4, #8231c4, #d93fbf, #d93f63, #d96a3f, #c9b52e, #6aa621, #299e2d);
    background-size: 400% 400%;
    -webkit-background-clip: text;
    background-clip: text;
    -webkit-text-fill-color: transparent;
}

.version.versionEasterEgg {
    animation: gradient .5s linear infinite, versionEasterEgg 3s linear infinite;
    transition: .3s cubic-bezier(.1, 0, 0, 1);
}

@keyframes gradient {
    0% {
        background-position: 0% 50%;
    }
    50% {
        background-position: 100% 50%;
    }
    100% {
        background-position: 0% 50%;
    }
}

@keyframes versionEasterEgg {
    0% {
        transform: scale(3) rotate(0deg);
    }
    50% {
        transform: scale(.5) rotate(360deg);
    }
    100% {
        transform: scale(3) rotate(720deg);
    }
}

h3 {
    margin-bottom: 0;
}

ul {
    margin-left: -10px;
}
</style>