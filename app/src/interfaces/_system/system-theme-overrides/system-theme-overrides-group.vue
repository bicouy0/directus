<script setup lang="ts">
import { isPlainObject } from 'lodash';
import { computed, ref } from 'vue';
import SystemThemeOverridesRule from './system-theme-overrides-rule.vue';
import type { SetValueFn } from './types.js';

defineOptions({
	name: 'SystemThemeOverridesGroup',
});

const props = defineProps<{
	rules: Record<string, unknown>;
	group?: string;
	root?: boolean;
	value: Record<string, unknown> | undefined;
	path: string[];
	set: SetValueFn;
}>();

const collapsed = ref(true);

const rulesGrouped = computed(() => {
	if (!props.root) return props.rules;

	return {
		$root: Object.fromEntries(Object.entries(props.rules).filter(([_key, value]) => isPlainObject(value) === false)),
		...Object.fromEntries(Object.entries(props.rules).filter(([_key, value]) => isPlainObject(value))),
	};
});

const hasValue = computed(() => {
	return !!props.value && (isPlainObject(props.value) ? Object.keys(props.value).length > 0 : true);
});
</script>

<template>
	<div class="theme-overrides-group" :class="{ root }">
		<button
			v-if="!root"
			class="group-toggle"
			:class="{ collapsed, 'has-value': hasValue }"
			@click="collapsed = !collapsed"
		>
			<span>{{ group ?? 'globals' }}</span>
			<v-icon class="icon" name="expand_more" small />
		</button>

		<transition-expand>
			<div v-if="root || !collapsed" class="group-contents">
				<template v-for="(ruleValue, ruleKey) in rulesGrouped" :key="ruleKey">
					<system-theme-overrides-group
						v-if="isPlainObject(ruleValue)"
						:group="ruleKey === '$root' ? undefined : ruleKey"
						:rules="(ruleValue as Record<string, unknown>)"
						:value="ruleKey === '$root' ? value : (value?.[ruleKey] as Record<string, unknown> | undefined)"
						:set="set"
						:path="ruleKey === '$root' ? path : [...path, ruleKey]"
					/>

					<system-theme-overrides-rule
						v-else
						:rule="ruleKey"
						type="color"
						:set="set"
						:default-value="(ruleValue as string | number)"
						:value="(value?.[ruleKey] as string | number | undefined)"
						:path="[...path, ruleKey]"
					/>
				</template>
			</div>
		</transition-expand>
	</div>
</template>

<style scoped lang="scss">
.theme-overrides-group {
	&:not(.root) {
		.group-contents {
			padding-left: 2ch;
			border-left: 1px solid var(--theme--border-color-subdued);
		}
	}
}

.group-toggle {
	font-family: var(--theme--font-family-monospace);
	color: var(--theme--form--field--input--foreground);
	width: calc(100% + 16px);
	text-align: left;
	padding-inline: 8px;
	margin-inline-start: -8px;

	&.has-value {
		position: relative;

		&::before {
			content: '';
			width: 4px;
			height: 4px;
			background-color: var(--theme--form--field--input--foreground-subdued);
			border-radius: 4px;
			position: absolute;
			top: 11px;
			left: -1px;
			display: block;
		}
	}

	&:hover {
		background-color: var(--background-subdued);
		border-radius: var(--theme--border-radius);
	}

	.icon {
		rotate: 0deg;
		transition: rotate var(--fast) var(--transition);
	}

	&.collapsed {
		.icon {
			rotate: -90deg;
		}
	}
}
</style>
