<template>
	<div v-if="prev || next" class="page-nav">
		<p class="page-nav__container">
			<span v-if="prev" class="page-nav__prev">
				<Button class="page-nav__prev-link" :link="prev.path">
					{{ prev.title || prev.path }}
				</Button>
			</span>

			<span v-if="next" class="page-nav__next">
				<Button class="page-nav__next-link" :link="next.path">
					{{ next.title || next.path }}
				</Button>
			</span>
		</p>
	</div>
</template>

<script>
import { resolvePage } from "../util";
import isString from "lodash/isString";
import isNil from "lodash/isNil";

export default {
	name: "PageNav",
	components: {
		Button: () => import("../../../components/general/Button")
	},
	props: ["sidebarItems"],

	computed: {
		prev() {
			return resolvePageLink(LINK_TYPES.PREV, this);
		},

		next() {
			return resolvePageLink(LINK_TYPES.NEXT, this);
		}
	}
};

function resolvePrev(page, items) {
	return find(page, items, -1);
}

function resolveNext(page, items) {
	return find(page, items, 1);
}

const LINK_TYPES = {
	NEXT: {
		resolveLink: resolveNext,
		getThemeLinkConfig: ({ nextLinks }) => nextLinks,
		getPageLinkConfig: ({ frontmatter }) => frontmatter.next
	},
	PREV: {
		resolveLink: resolvePrev,
		getThemeLinkConfig: ({ prevLinks }) => prevLinks,
		getPageLinkConfig: ({ frontmatter }) => frontmatter.prev
	}
};

function resolvePageLink(
	linkType,
	{ $themeConfig, $page, $route, $site, sidebarItems }
) {
	const { resolveLink, getThemeLinkConfig, getPageLinkConfig } = linkType;

	// Get link config from theme
	const themeLinkConfig = getThemeLinkConfig($themeConfig);

	// Get link config from current page
	const pageLinkConfig = getPageLinkConfig($page);

	// Page link config will overwrite global theme link config if defined
	const link = isNil(pageLinkConfig) ? themeLinkConfig : pageLinkConfig;

	if (link === false) {
		return;
	} else if (isString(link)) {
		return resolvePage($site.pages, link, $route.path);
	} else {
		return resolveLink($page, sidebarItems);
	}
}

function find(page, items, offset) {
	const res = [];
	flatten(items, res);
	for (let i = 0; i < res.length; i++) {
		const cur = res[i];
		if (cur.type === "page" && cur.path === decodeURIComponent(page.path)) {
			return res[i + offset];
		}
	}
}

function flatten(items, res) {
	for (let i = 0, l = items.length; i < l; i++) {
		if (items[i].type === "group") {
			flatten(items[i].children || [], res);
		} else {
			res.push(items[i]);
		}
	}
}
</script>
