<template>
    <div class="Liberty" :style="skinConfig">
        <div id="top"></div>
        <div class="nav-wrapper" :class="{ 'navbar-fixed-top': $store.state.localConfig['liberty-prime.fixed_navbar'] === true }">
            <nav class="navbar navbar-dark">
                <nuxt-link class="navbar-brand" to="/">{{ libertyConfig('navbar_logo_text') ?? $store.state.config['logo_text'] }}</nuxt-link>
                <ul class="nav navbar-nav">
                    <li class="nav-item">
                        <nuxt-link class="nav-link" to="/RecentChanges"><span class="fa fa-refresh"></span><span class="hide-title">최근 변경</span></nuxt-link>
                    </li>
                    <li class="nav-item">
                        <nuxt-link class="nav-link" to="/RecentDiscuss"><span class="fa fa-comments"></span><span class="hide-title">최근 토론</span></nuxt-link>
                    </li>
                    <li class="nav-item">
                        <nuxt-link class="nav-link" to="/random"><span class="fa fa-random"></span><span class="hide-title">임의 문서</span></nuxt-link>
                    </li>
                    <li class="nav-item">
                        <dropdown>
                            <template #toggle>
                                <a class="nav-link dropdown-toggle dropdown-toggle-fix" href="#" @click.prevent>
                                    <span class="fa fa-gear"></span><span class="hide-title">도구</span>
                                </a>
                            </template>
                            <div class="dropdown-menu" role="menu">
                                <nuxt-link to="/Upload" class="dropdown-item">파일 올리기</nuxt-link>
                                <div class="dropdown-divider"></div>
                                <nuxt-link to="/NeededPages" class="dropdown-item">작성이 필요한 문서</nuxt-link>
                                <nuxt-link to="/OrphanedPages" class="dropdown-item">고립된 문서</nuxt-link>
                                <nuxt-link to="/OrphanedCategories" class="dropdown-item">고립된 분류</nuxt-link>
                                <nuxt-link to="/UncategorizedPages" class="dropdown-item">분류가 되지 않은 문서</nuxt-link>
                                <nuxt-link to="/OldPages" class="dropdown-item">편집된 지 오래된 문서</nuxt-link>
                                <nuxt-link to="/ShortestPages" class="dropdown-item">내용이 짧은 문서</nuxt-link>
                                <nuxt-link to="/LongestPages" class="dropdown-item">내용이 긴 문서</nuxt-link>
                                <nuxt-link to="/BlockHistory" class="dropdown-item">차단 내역</nuxt-link>
                                <nuxt-link to="/RandomPage" class="dropdown-item">RandomPage</nuxt-link>
                                <nuxt-link to="/License" class="dropdown-item">라이선스</nuxt-link>
                                <template v-if="$store.state.session.menus.length">
                                    <div class="dropdown-divider"></div>
                                    <nuxt-link v-for="m in $store.state.session.menus" :key="m.l" :to="m.l" class="dropdown-item">{{ m.t }}</nuxt-link>
                                    <nuxt-link v-if="$store.state.session.menus.some(item => item.l === '/admin/developer') && ! $store.state.session.menus.some(item => item.l === '/admin/initial_setup')" class="dropdown-item" to="/admin/initial_setup">초기 설정</nuxt-link>
                                </template>
                            </div>
                        </dropdown>
                    </li>
                </ul>
                <div class="navbar-login">
                    <dropdown class="login-menu">
                        <template #toggle>
                            <a id="login-menu" class="dropdown-toggle" type="button">
                                <img v-if="$store.state.session.gravatar_url" class="profile-img" :src="$store.state.session.gravatar_url">
                                <span v-else class="fa fa-user"></span>
                            </a>
                        </template>
                        <div class="dropdown-menu dropdown-menu-right login-dropdown-menu">
                            <div v-if="$store.state.session.account.type === 1" class="username dropdown-item">
                                <b>{{ $store.state.session.account.name }}</b><br>Member
                            </div>
                            <div v-else-if="$store.state.session.account.type === 0" class="username dropdown-item">
                                <b>{{ $store.state.session.account.name }}</b><br>Please login!
                            </div>
                            <template v-if="$store.state.session.account.type === 1">
                                <nuxt-link
                                    v-for="n in $store.state.session.otherAccounts"
                                    :key="n.uuid"
                                    :to="`/member/switch_account/${n.uuid}`"
                                    class="dropdown-item switch-account-item"
                                >
                                    <img :src="n.gravatar_url" class="switch-account-image" />
                                    <span class="switch-account-name">{{ n.name }}</span>
                                    <button
                                        type="button"
                                        class="switch-account-logout"
                                        @click.prevent="logoutOther(n.uuid)"
                                        :aria-label="`로그아웃 ${n.name}`"
                                    >
                                        <span class="fa-solid fa-x" />
                                    </button>
                                </nuxt-link>
                                <nuxt-link to="/member/login" class="dropdown-item">계정 추가</nuxt-link>
                            </template>
                            <div class="dropdown-divider"></div>
                            <a href="#" class="dropdown-item" @click.prevent="openSettingModal">설정</a>
                            <a v-if="$store.state.currentTheme === 'light'" href="#" class="dropdown-item" @click.prevent="$store.commit('localConfigSetValue', {key: 'wiki.theme', value: 'dark'})">다크 테마로</a>
                            <a v-if="$store.state.currentTheme === 'dark'" href="#" class="dropdown-item" @click.prevent="$store.commit('localConfigSetValue', {key: 'wiki.theme', value: 'light'})">라이트 테마로</a>
                            <div class="dropdown-divider"></div>
                            <template v-if="$store.state.session.account.type === 1">
                                <nuxt-link to="/member/mypage" class="dropdown-item">내 정보</nuxt-link>
                                <nuxt-link :to="doc_action_link(user_doc($store.state.session.account.name), 'w')" class="dropdown-item">내 사용자 문서</nuxt-link>
                                <nuxt-link to="/member/starred_documents" class="dropdown-item">내 문서함</nuxt-link>
                                <div class="dropdown-divider"></div>
                            </template>
                            <template v-if="$store.state.session.account.uuid">
                                <nuxt-link class="dropdown-item" :to="contribution_link($store.state.session.account.uuid)">내 문서 기여 목록</nuxt-link>
                                <nuxt-link class="dropdown-item" :to="contribution_link_discuss($store.state.session.account.uuid)">내 토론 기여 목록</nuxt-link>
                                <nuxt-link class="dropdown-item" :to="contribution_link_edit_request($store.state.session.account.uuid)">내 편집 요청 목록</nuxt-link>
                                <div class="dropdown-divider"></div>
                            </template>
                            <nuxt-link v-if="$store.state.session.account.type === 1" :to="{path:'/member/logout',query:{redirect:$route.fullPath}}" class="dropdown-item">로그아웃</nuxt-link>
                            <nuxt-link v-else :to="{path:'/member/login',query:{redirect:$route.fullPath}}" class="dropdown-item">로그인</nuxt-link>
                        </div>
                    </dropdown>
                </div>
                <div class="nav-link navbar-notification">
                    <dropdown class="liberty-notification-menu">
                        <template #toggle>
                            <a class="nav-link dropdown-toggle liberty-notification-toggle" href="#" @click.prevent>
                                <span class="fa fa-bell"></span>
                                <span v-if="notifications.length" class="liberty-notification-count">{{ notificationCountLabel }}</span>
                            </a>
                        </template>
                        <div class="dropdown-menu dropdown-menu-right liberty-notification-dropdown" role="menu">
                            <div class="liberty-notification-header">
                                <strong>알림</strong>
                                <div class="liberty-notification-actions">
                                    <a v-if="notifications.length" href="#" class="liberty-notification-action" @click.prevent="markAllNotificationsRead">모두 읽기</a>
                                    <nuxt-link to="/member/notifications" class="liberty-notification-action">자세히</nuxt-link>
                                </div>
                            </div>
                            <div v-if="!notifications.length" class="liberty-notification-empty">새로운 알림이 없습니다.</div>
                            <nuxt-link
                                v-for="item in notificationPreviewItems"
                                :key="item.uuid || `${item.type}-${item.createdAt}`"
                                :to="notificationLink(item)"
                                class="dropdown-item liberty-notification-item"
                                :class="{ 'is-read': item.read }"
                            >
                                <span class="liberty-notification-item-icon" :class="`type-${item.type}`">
                                    <span :class="notificationTypeIcon(item.type)"></span>
                                </span>
                                <span class="liberty-notification-item-content">
                                    <span class="liberty-notification-item-title">{{ notificationTitle(item) }}</span>
                                    <span v-if="notificationDetail(item)" class="liberty-notification-item-detail">{{ notificationDetail(item) }}</span>
                                </span>
                            </nuxt-link>
                        </div>
                    </dropdown>
                </div>
                <search-form />
            </nav>
        </div>
        <div class="content-wrapper" :class="{ 'hide-sidebar': $store.state.localConfig['liberty-prime.sidebar'] === 'hide' || $store.state.localConfig['liberty-prime.sidebar'] === 'footer' }">
            <div class="liberty-sidebar">
                <div class="liberty-right-fixed" :class="{ 'fixed': $store.state.localConfig['liberty-prime.sidebar'] === 'fix' }">
                    <div class="live-recent">
                        <div class="live-recent-header">
                            <ul class="nav nav-tabs">
                                <li class="nav-item">
                                    <a id="liberty-recent-tab1" class="nav-link active">최근 변경</a>
                                </li>
                            </ul>
                        </div>
                        <recent-card />
                        <div class="live-recent-footer">
                            <nuxt-link to="/RecentChanges" title="최근 변경내역"><span class="label label-info">더 보기</span></nuxt-link>
                        </div>
                    </div>
                </div>
            </div>
            <div class="container-fluid liberty-content">
                <div v-if="libertyConfig('wiki.sitenotice')" id="site-notice" class="notification">
                    <span class="label" v-html="libertyConfig('wiki.sitenotice')" @click="onDynamicContentClick($event)" />
                </div>
                <div class="liberty-content-header">
                    <content-tool @onClickEditBtn="showEditMessage" />
                    <div class="title">
                        <h1 v-if="$store.state.page.data.document && $store.state.page.viewName !== 'error'">
                            <nuxt-link :to="doc_action_link($store.state.page.data.document, 'w')"><span v-if="$store.state.page.data.document.forceShowNamespace !== false" class="namespace">{{$store.state.page.data.document.namespace}}:</span>{{$store.state.page.data.document.title}}</nuxt-link>
                            <small v-if="$store.state.page.viewName === 'edit_edit_request' || $store.state.page.viewName === 'edit_request'">(편집 요청)</small>
                            <small v-else-if="$store.state.page.viewName === 'edit' && $store.state.page.data.body.section">(r{{$store.state.page.data.body.baserev}} 문단 편집)</small>
                            <small v-else-if="$store.state.page.viewName === 'edit' && $store.state.page.data.body.baserev === '0'">(새 문서 생성)</small>
                            <small v-else-if="$store.state.page.viewName === 'edit'">(r{{$store.state.page.data.body.baserev}} 편집)</small>
                            <small v-else-if="$store.state.page.viewName === 'history'">(역사)</small>
                            <small v-else-if="$store.state.page.viewName === 'backlink'">(역링크)</small>
                            <small v-else-if="$store.state.page.viewName === 'move'">(이동)</small>
                            <small v-else-if="$store.state.page.viewName === 'delete'">(삭제)</small>
                            <small v-else-if="$store.state.page.viewName === 'acl'">(ACL)</small>
                            <small v-else-if="$store.state.page.viewName === 'thread'">(토론)</small>
                            <small v-else-if="$store.state.page.viewName === 'thread_list'">(토론 목록)</small>
                            <small v-else-if="$store.state.page.viewName === 'thread_list_close'">(닫힌 토론)</small>
                            <small v-else-if="$store.state.page.viewName === 'edit_request_close'">(닫힌 편집 요청)</small>
                            <small v-else-if="$store.state.page.viewName === 'diff'">(비교)</small>
                            <small v-else-if="$store.state.page.viewName === 'revert' && $store.state.page.data.rev">(r{{$store.state.page.data.rev}}로 되돌리기)</small>
                            <small v-else-if="$store.state.page.viewName === 'raw' && $store.state.page.data.rev">(r{{$store.state.page.data.rev}} RAW)</small>
                            <small v-else-if="$store.state.page.viewName === 'blame' && $store.state.page.data.rev">(r{{$store.state.page.data.rev}} Blame)</small>
                            <small v-else-if="$store.state.page.viewName === 'wiki' && $store.state.page.data.rev">(r{{$store.state.page.data.rev}} 판)</small>
                        </h1>
                        <h1 v-else>{{ $store.state.page.title }}</h1>
                    </div>
                </div>
                <div class="liberty-content-main wiki-article">
                    <alert v-if="isShowACLMessage && $store.state.page.data.edit_acl_message" @close="isShowACLMessage = false" error closable>
                        <span v-html="$store.state.page.data.edit_acl_message" @click="onDynamicContentClick($event)"></span>
                        <span v-if="requestable"><br v-if="$store.state.page.data.edit_acl_message.includes('\n')"> 대신 <nuxt-link :to="doc_action_link($store.state.page.data.document, 'new_edit_request')">편집 요청</nuxt-link>을 생성할 수 있습니다.</span>
                    </alert>
                    <alert v-if="$store.state.session.user_document_discuss && $store.state.localConfig['wiki.hide_user_document_discuss'] !== $store.state.session.user_document_discuss" @close="$store.commit('localConfigSetValue', {key: 'wiki.hide_user_document_discuss', value: $store.state.session.user_document_discuss})" closable theme="primary">
                        현재 진행 중인 <nuxt-link :to="doc_action_link(user_doc($store.state.session.account.name), 'discuss')">사용자 토론</nuxt-link>이 있습니다.
                    </alert>
                    <alert v-if="$store.state.page.viewName === 'notfound' && $store.state.page.data.document.namespace === '문서'" style="line-height: 2.1rem;">
                        '{{ $store.state.page.title }}'을(를) 검색하시겠습니까?
                        <div class="float-right"><seed-link-button :to="'/Search?q='+ $store.state.page.title">검색</seed-link-button></div>
                        <div class="clearfix"></div>
                    </alert>
                    <nuxt />
                    <div v-if="$store.state.page.viewName === 'license'">
                        <LicensePage />
                        <pre>{{ License }}</pre>
                    </div>
                    <div class="clearfix"></div>
                </div>
                <div id="bottom" class="liberty-footer">
                    <ul v-if="$store.state.page.viewName === 'wiki' && $store.state.page.data.date" class="footer-info">
                        <li v-if="$store.state.page.data.rev" class="footer-info-lastmod">이 리비전은 <local-date :date="$store.state.page.data.date" />에 편집되었습니다.</li>
                        <li v-else class="footer-info-lastmod">이 문서는 <local-date :date="$store.state.page.data.date" />에 마지막으로 편집되었습니다.</li>
                        <li class="footer-info-copyright" v-html="$store.state.page.data.copyright_text" />
                    </ul>
                    <ul class="footer-places" @click="onDynamicContentClick($event)" v-html="libertyConfig('footer_html') || libertyConfig('wiki.footer_text')" />
                    <ul class="footer-icons">
                        <li class="footer-poweredbyico">
                            <a href="//github.com/NINEAPPLE/LibertyPrime" target="_blank">LibertyPrime</a> | <a href="//github.com/wjdgustn/thetree" target="_blank">the tree</a>
                        </li>
                    </ul>
                </div>
                <div v-if="$store.state.localConfig['liberty-prime.sidebar'] === 'footer'" class="footer-recent">
                    <recent-card :limit="8" />
                    <div class="live-recent-footer">
                        <nuxt-link to="/RecentChanges" title="최근 변경내역"><span class="label label-info">더 보기</span></nuxt-link>
                    </div>
                </div>
            </div>
        </div>
        <div class="scroll-buttons">
            <nuxt-link class="scroll-toc" to="#toc"><i class="fa fa-list-alt" aria-hidden="true"></i></nuxt-link>
            <nuxt-link id="left" class="scroll-button" to="#top"><i class="fa fa-arrow-up" aria-hidden="true"></i></nuxt-link>
            <nuxt-link id="right" class="scroll-bottom" to="#bottom"><i class="fa fa-arrow-down" aria-hidden="true"></i></nuxt-link>
        </div>
    </div>
</template>

<style>
@import "./css/bootstrap.min.css";
@import "https://cdnjs.cloudflare.com/ajax/libs/font-awesome/7.0.1/css/all.min.css";
@import "./css/font-awesome.min.css";
@import "./css/font/Noto Sans KR.css";
@import "./css/default.css";
@import './css/default_mobile.css';
@import "./css/dark.css";
</style>

<script>
import Common from '~/mixins/common';
import Alert from '~/components/alert';
import SeedLinkButton from '~/components/seedLinkButton';
import LocalDate from '~/components/localDate';
import RecentCard from './layouts/recentCard';
import SearchForm from './layouts/searchForm';
import ContentTool from './layouts/contentTool';
import LicensePage from './layouts/license';
import Dropdown from './components/dropdown';
import SettingModal from './components/settingModal';
import License from "raw-loader!./LICENSE";

export default {
    mixins: [Common],
    components: {
        Alert,
        SeedLinkButton,
        LocalDate,
        RecentCard,
        SearchForm,
        LicensePage,
        Dropdown,
        ContentTool
    },
    data() {
        return {
            License,
            isShowACLMessage: false
        };
    },
    watch: {
        $route() {
            this.isShowACLMessage = false;
        }
    },
    head() {
        return {
            meta: [{ name: 'theme-color', content: this.brand_color }]
        };
    },
    computed: {
        brand_color() {
            return this.selectByTheme(this.libertyConfig('brand_color_1') ?? this.libertyConfig('theme_color') ?? '#4188f1', '#2d2f34');
        },
        skinConfig() {
            return {
                '--liberty-brand-color': this.brand_color,
                '--liberty-brand-dark-color': this.selectByTheme(this.libertyConfig('brand_dark_color_1') ?? this.darkenColor(this.brand_color), '#16171a'),
                '--liberty-brand-bright-color': this.selectByTheme(this.libertyConfig('brand_bright_color_1') ?? this.lightenColor(this.brand_color), '#383b40'),
                '--liberty-navbar-logo-image': this.libertyConfig('navbar_logo_image') || (this.libertyConfig('wiki.logo_url') && `url(${this.libertyConfig('wiki.logo_url')})`),
                '--liberty-navbar-logo-minimum-width': this.libertyConfig('navbar_logo_minimum_width'),
                '--liberty-navbar-logo-width': this.libertyConfig('navbar_logo_width'),
                '--liberty-navbar-logo-size': this.libertyConfig('navbar_logo_size'),
                '--liberty-navbar-logo-padding': this.libertyConfig('navbar_logo_padding'),
                '--liberty-navbar-logo-margin': this.libertyConfig('navbar_logo_margin'),
                '--brand-color-1': 'var(--liberty-brand-color)',
                '--brand-color-2': this.selectByTheme(this.libertyConfig('brand_color_2') ?? 'var(--liberty-brand-color)', 'var(--liberty-brand-color)'),
                '--brand-bright-color-1': 'var(--liberty-brand-bright-color)',
                '--brand-bright-color-2': this.selectByTheme(this.libertyConfig('brand_bright_color_2') ?? 'var(--liberty-brand-bright-color)', 'var(--liberty-brand-bright-color)'),
                '--text-color': this.selectByTheme('#373a3c', '#ddd'),
                '--article-background-color': this.selectByTheme('#fff', '#000'),
            };
        },
        requestable() {
            return this.$store.state.page.data.editable === true && this.$store.state.page.data.edit_acl_message && this.$store.state.page.viewName !== 'notfound';
        },
        notifications() {
            return Array.isArray(this.$store.state.session.notifications) ? this.$store.state.session.notifications : [];
        },
        notificationPreviewItems() {
            return this.notifications.slice(0, 8);
        },
        notificationCountLabel() {
            if (this.notifications.length > 99) return '99+';
            return String(this.notifications.length);
        }
    },
    methods: {
        showEditMessage() {
            if (this.isShowACLMessage) {
                this.$router.push(this.doc_action_link(this.$store.state.page.data.document, this.requestable ? 'new_edit_request' : 'edit'));
            }
            else {
                this.isShowACLMessage = true;
            }
        },
        darkenColor(hex, percent=50) {
            let r = parseInt(hex.substring(1, 3), 16);
            let g = parseInt(hex.substring(3, 5), 16);
            let b = parseInt(hex.substring(5, 7), 16);

            r = Math.round(r * (1 - percent / 100));
            g = Math.round(g * (1 - percent / 100));
            b = Math.round(b * (1 - percent / 100));

            return "#" + ((r < 16 ? "0" : "") + r.toString(16)) + ((g < 16 ? "0" : "") + g.toString(16)) + ((b < 16 ? "0" : "") + b.toString(16));
        },
        lightenColor(hex, percent=50) {
            let r = parseInt(hex.substring(1, 3), 16);
            let g = parseInt(hex.substring(3, 5), 16);
            let b = parseInt(hex.substring(5, 7), 16);

            r = Math.round(r + (255 - r) * (percent / 100));
            g = Math.round(g + (255 - g) * (percent / 100));
            b = Math.round(b + (255 - b) * (percent / 100));

            return "#" + ((r < 16 ? "0" : "") + r.toString(16)) + ((g < 16 ? "0" : "") + g.toString(16)) + ((b < 16 ? "0" : "") + b.toString(16));
        },
        libertyConfig(key) {
            if (key.startsWith('wiki.')) {
                return this.$store.state.config[key];
            }
            return this.$store.state.config[`skin.${__THETREE_SKIN_NAME__}.${key}`];
        },
        notificationLink(item) {
            return (item && item.url) || '/member/notifications';
        },
        notificationTypeIcon(type) {
            return ({
                0: 'fa fa-comments',
                1: 'fa fa-at',
                2: 'fa fa-bullhorn',
                3: 'fa fa-bell'
            })[type] || 'fa fa-bell';
        },
        notificationTitle(item) {
            const username = item && item.comment && item.comment.user && item.comment.user.name;
            const thread = item && item.thread && item.thread.topic;
            const commentId = item && item.comment && item.comment.id;

            if (item && item.type === 0) {
                const by = username ? `${username} 사용자가 ` : '';
                const topic = thread ? `${thread}` : '토론';
                return `${by}${topic}${commentId ? ` #${commentId}` : ''} 사용자 토론 댓글 작성`;
            }

            if (item && item.type === 1) {
                const by = username ? `${username} 사용자가 ` : '';
                const topic = thread ? `${thread}` : '토론';
                return `${by}${topic}${commentId ? ` #${commentId}` : ''} 댓글에서 호출`;
            }

            if (item && item.type === 2) {
                return '운영 공지';
            }

            if (item && item.type === 3) {
                return '알림';
            }

            return '새 알림';
        },
        notificationDetail(item) {
            if (!item) return '';

            if (item.type === 2 || item.type === 3) {
                return this.stripHtml(item.data);
            }

            if (item.comment && item.comment.contentHtml) {
                return this.stripHtml(item.comment.contentHtml);
            }

            if (item.document) {
                return this.doc_fulltitle(item.document);
            }

            return '';
        },
        stripHtml(raw) {
            if (!raw) return '';
            return String(raw).replace(/<[^>]*>/g, ' ').replace(/\s+/g, ' ').trim();
        },
        async markAllNotificationsRead() {
            if (!this.notifications.length) return;

            await this.internalRequestAndProcess('/member/notifications/read', {
                method: 'POST'
            });

            this.$store.state.session.notifications = [];
        },
        selectByTheme(light, dark) {
            return this.$store.state.currentTheme === 'dark' ? dark : light;
        },
        openSettingModal() {
            this.$vfm.show({ component: SettingModal });
        },
        async logoutOther(uuid) {
            if (!uuid) return;

            try {
                await this.internalRequestAndProcess(`/member/logout_other/${uuid}`, {
                    method: 'POST'
                });
            } catch (err) {
                console.error('logoutOther failed', err);
            } finally {
                // Ensure UI updates to reflect session changes
                if (typeof window !== 'undefined') {
                    window.location.reload();
                }
            }
        },
    }
}
</script>
