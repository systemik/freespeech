<template>
	<v-dialog
		v-model="editDialogVisibility"
		persistent
		max-width="600px"
	>
		<v-card>
			<v-card-title>
				<span class="headline"> <v-icon>{{ currentTileBeingEdited.newTile ? 'add' : 'edit' }}</v-icon>{{ currentTileBeingEdited.newTile ? "Add a new tile" : "Edit" }}</span>
			</v-card-title>
			<v-card-text>
				<v-container>
					<v-form
						ref="form"
						id="editTileForm"
						v-model="valid"
					>
						<v-row >

							<v-col cols="12">
								<v-text-field
									:value="currentLocaleTileBeingEdited.name"
									:label="$t('editMode.tile.name')"
									@input="update('name', $event)"
									:rules="[rules.required]"
								/>
								<v-text-field
									:value="currentLocaleTileBeingEdited.image"
									:label="$t('editMode.tile.imageUrl')"
									@input="update('image', $event)"
								/>
								<v-img
									max-height="32"
									max-width="32"
									aspect-ratio="1"
									:src="currentLocaleTileBeingEdited.image"
								/>
								<v-text-field
									class="pt-5"
									:value="currentLocaleTileBeingEdited.text"
									:label="$t('editMode.tile.textToSpeak')"
									@input="update('text', $event)"
									:rules="[rules.required]"
								/>
								<p class="text-left">
									{{ $t('editMode.tile.accentColor') }}
								</p>
								<v-select
									:items="tileColors"
									v-model="currentLocaleTileBeingEdited.accent"
									filled
									label="Accent"
								></v-select>

								<p class="text-left" >
									{{ $t('editMode.tile.navigation') }}
								</p>

								<v-text-field
									:value="currentLocaleTileBeingEdited.navigation"
									:label="$t('editMode.tile.navigationLabel')"
									@input="update('navigation', $event)"
								/>

							</v-col>

						</v-row>
					</v-form>

				</v-container>
			</v-card-text>
			<v-card-actions>
				<v-spacer />
				<v-btn
					color="blue darken-1"
					text
					@click="saveCurrentEdit"
				>
					{{ $t('editMode.tile.buttonSave') }}
				</v-btn>

				<v-btn
					color="blue darken-1"
					text
					@click="toggleEditDialogVisibility"
				>
					{{ $t('editMode.tile.buttonClose') }}
				</v-btn>
			</v-card-actions>
		</v-card>
	</v-dialog>
</template>

<script>
import { mapActions, mapGetters } from 'vuex';

export default {
	...mapGetters('tilePad', [
		'currentTileBeingEdited',
	]),
	name: 'EditTileDialog',
	data () {
		return {
			tileColors: ['blush', 'peach', 'pear', 'mint', 'violet', 'white'],
			currentColor: 'peach',
			valid:false,
			color: '',
			newTileObject: {},
			rules: {
				required: value => !!value || 'This field is required.'
			}
		};
	},
	computed: {
		...mapGetters('tilePad', [
			'editDialogVisibility',
			'currentTileBeingEdited',
			'customTilePadData'
		]),
		areRequiredFieldsFilled:function(){
			return !!this.newTileObject.name && !!this.newTileObject.text;
		},
		navigations: function (){
			return  Object.keys(this.customTilePadData);
		},
		currentLocaleTileBeingEdited: function (){
			const messages = this.currentTileBeingEdited.messages;
			//console.log('editTileDialog.vue - newTileObject:',this.currentTileBeingEdited);
			//console.log('editTileDialog.vue - newTileObjectmessages:',messages);
			const currentLocale = Object.keys(this.$root.$i18n._localeChainCache)[0];
			//console.log('editTileDialog.vue - newTileObjectlocale:',currentLocale);
			if (
				typeof messages !== 'undefined' &&
        typeof messages[currentLocale] === 'object'
			) {
				console.log('editTileDialog.vue - replaced by locale');
				return { ...this.currentTileBeingEdited,
					...messages[currentLocale] };
			} else {
				console.log('editTileDialog.vue - not replaced by locale');
				return this.currentTileBeingEdited;
			}
		}

	},
	methods: {
		...mapActions('tilePad', [
			'toggleEditDialogVisibility',
			'saveEditsToTileBeingEdited',
			'createNewTile',
			'saveTileEdit']),
		validate () {
			this.$refs.form.validate();
		},
		resetValidation () {
			this.$refs.form.resetValidation();
		},
		saveCurrentEdit () {

			//this.$root.$i18n.locale = this.locale;
			//console.log(this.$root.$i18n._localeChainCache.fr[0]);
			//console.log(Object.keys(this.$root.$i18n._localeChainCache)[0]);
			//console.log(this.localeTileData);
			//console.log(this.currentTileBeingEdited);
			if(this.currentTileBeingEdited.newTile){
				this.validate();

				if(this.valid)
				{
					this.resetValidation();
					var maxid = 0;

					this.newTileObject.id = maxid + 1;
					delete this.currentTileBeingEdited.newTile;
					this.createNewTile(this.newTileObject);
					this.toggleEditDialogVisibility();
				}
				return;
			}else{

				this.saveTileEdit();
				this.toggleEditDialogVisibility();
			}
		},
		update (key, value ) {
			if(!this.currentTileBeingEdited.newTile){
				this.saveEditsToTileBeingEdited({
					'key': key,
					'value': value
				});
			}else{
				this.newTileObject[key] = value;
			}
		},
		onBlurUpdate(key, value){
			if(!this.currentTileBeingEdited.newTile){
				this.saveEditsToTileBeingEdited({
					'key': key,
					'value': value.target.value
				});
			}else{
				this.newTileObject[key] = value;
			}
		}
	},
};
</script>

<style>

</style>
