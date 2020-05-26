

<!-- components/FretboardTools.vue -->


<!--{{{ Pug -->
<template lang="pug">

div.FretboardTools
	//- Instrument settings
	component(
		:is="isMobileDevice ? 'div' : 'VPopupMenu'"

		icon="guitar"
		title="Instrument settings"
		)
		div.instrument-settings
			div.instrument-settings__main
				//- Instrument
				VSelect(
					:options="instruments"
					v-model="instrument"
					)
				//- Tuning
				VSelect(
					:options="tunings"
					v-model="tuning"
					)
			//- Fret range
			div.instrument-settings__fret-range
				VMultiRange.fret-range__slider(
					:min="0"
					:max="MAX_NB_FRETS - 1"
					:min-gap="MIN_NB_FRETS"
					:is-flipped="isFlipped"

					:values="fretRangeDisplay"

					@input=" fretRangeDisplay = $event"
					@change="fretRange = fretRangeDisplay = $event"
					)
				p.fret-range__text
					span(v-html="formatFretNb(fretRangeDisplay[isFlipped ? 1 : 0])")
					span.fret-range__text__separator —
					span(v-html="formatFretNb(fretRangeDisplay[isFlipped ? 0 : 1])")

			//- Switch fretting hand
			VButton(
				icon="hand-paper"
				title="Switch fretting hand"

				:is-flipped="!isFlipped"
				@click="$store.commit('fretboard/toggle.isFlipped')"
				)

	//- Toggle fret numbers
	VButton(
		:icon="['fal', 'list-ol']"
		title="Show numbers"

		:is-active="isShowingFretNbs"
		@click="$store.commit('fretboard/toggle.isShowingFretNbs')"
		)

	//- Toggle note names
	VButton(
		icon="info-circle"
		title="Show notes"

		:is-active="isShowingNoteNames"
		@click="$store.commit('fretboard/toggle.isShowingNoteNames')"
		)

	//- Export the fretboard
	VPopupMenu(
		v-if="!isMobileDevice"

		icon="file-download"
		title="Export"

		:force-closing="exportMenuClose"
		)
		div.export-menu
			div.export-menu__text
				p: strong Choose a format to export in
				p: em
					| If you want to print this  fretboard, choose PNG or JPG.
					| The SVG format is most useful for embedding in web pages as it will scale perfectly when resized.
			div.export-menu__buttons
				each format of ['png', 'jpg', 'svg']
					VButton(
						icon=(format == 'svg' ? 'file-image' : 'image-polaroid')
						title=format.toUpperCase()

						@click=`exportFretboard('${format}')`
						)

</template>
<!--}}}-->


<!--{{{ JavaScript -->
<script>

import { get, sync }                          from 'vuex-pathify'

import { MIN_NB_FRETS, MAX_NB_FRETS }         from '@/modules/constants'
import { mapObjectToObject }                  from '@/modules/object'
import { exportFretboard }                    from '@/modules/export'
import { instruments, tunings, tuningsNames } from '@/modules/music'

export default {
	name: 'FretboardTools',

	data() {
		return {
			exportMenuClose:  false,
			fretRangeDisplay: this.$store.state.fretboard.fretRange,
		}
	},

	computed: {
		tunings()
		{
			return mapObjectToObject(tunings[this.instrument], tuning => tuningsNames[tuning]);
		},
		...sync('fretboard', [
			'instrument',
			'tuning',
			'fretRange',
			'isShowingFretNbs',
			'isShowingNoteNames',
			'isFlipped',
		]),
		...get([
			'isDarkModeOn',
			'isMobileDevice',
		]),
	},

	created()
	{
		this.instruments  = instruments;
		this.MIN_NB_FRETS = MIN_NB_FRETS;
		this.MAX_NB_FRETS = MAX_NB_FRETS;
	},

	methods: {
		exportFretboard(format)
		{
			this.exportMenuClose = !this.exportMenuClose;

			exportFretboard(
				format,
				this.$store.state.sequences.sequences,
				this.$store.getters['sequences/displayedSequences'],
				tunings[this.instrument][this.tuning],
				instruments[this.instrument].nbStrings,
				this.fretRange[0],
				this.fretRange[1],
				this.isFlipped,
				this.isShowingNoteNames,
				this.isShowingFretNbs,
				this.isDarkModeOn,
				format != 'svg',
			);
		},
		formatFretNb(number)
		{
			if (number == 0)
				return 'Open strings';

			if (11 <= number && number <= 13)
				return `${number}<sup>th</sup> fret`;

			switch (`${number}`.slice(-1))
			{
				case '1': return `${number}<sup>st</sup> fret`;
				case '2': return `${number}<sup>nd</sup> fret`;
				case '3': return `${number}<sup>rd</sup> fret`;
				default:  return `${number}<sup>th</sup> fret`;
			}
		},
	}
}

</script>
<!--}}}-->


<!--{{{ SCSS -->
<style lang="scss" scoped>

.FretboardTools {
	@include mq($until: desktop)
	{
		@include center-column;
		@include space-children-v(40px);
	}

	@include mq($from: desktop)
	{
		display: flex;
		@include space-children-h(10px);
	}
}

.instrument-settings {
	@include center-column;

	@include mq($until: desktop)
	{
		@include space-children-v(40px);
	}

	@include mq($from: desktop)
	{
		@include space-children-v(20px);

		padding: 10px;
	}
}

.instrument-settings__main {
	display: flex;
	@include space-children-h(10px);
}

.instrument-settings__fret-range {
	@include space-children-v(4px);
}

.fret-range__slider {
	min-width: 250px;
}

.fret-range__text {
	text-align: center;
	@include space-children-h(6px);

	color: var(--color--text);
}

.fret-range__text__separator {
	color: var(--color--text--secondary);
}

.export-menu {
	@include space-children-v(20px);

	padding: 10px;
}

.export-menu__text {
	@include space-children-v(10px);

	max-width: 300px;

	font-size: 1.6rem;
	text-align: justify;

	color: var(--color--text);
}

.export-menu__buttons {
	display: flex;
	justify-content: center;
	@include space-children-h(10px);
}

</style>
<!--}}}-->