<template>
    <div>
        <label class="control-label" :for="options.label || property.name" v-text="options.label || property.name"></label>
        <br class="clearfix">
        <div v-if="displayMode === 'EDIT' || displayMode === 'CREATE'">
            <div v-if="typeof items[0] === 'object'" class="single">
                <multiselect v-model="selected"
                    :options="items"
                    open-direction="bottom"
                    track-by="_id"
                    :label="displayField || 'Name'"
                    :searchable="!!options.searchable"
                    :close-on-select="true"
                    select-label=""
                    deselect-label=""
                    :hide-selected="true"
                    :disabled="options.disabled"
                    @close="handler">
                    <template slot="option" slot-scope="props">
                        <div class="image" v-if="props.option._ImageUrl" :style="{'background-image': 'url(' + props.option._ImageUrl + ')'}"></div>
                        <div class="description">
                            <div class="title">
                                \{{ props.option.Name }}
                            </div>
                            <small class="text-muted" v-if="props.option.Description">
                                \{{ props.option.Description }}
                            </small>
                        </div>
                    </template>
                </multiselect>
            </div>
            <div v-else class="single">
                <multiselect v-model="selected"
                    :options="items"
                    open-direction="bottom"
                    :searchable="!!options.searchable"
                    :close-on-select="true"
                    select-label=""
                    deselect-label=""
                    :disabled="options.disabled"
                    :hide-selected="true"
                    @close="handler">
                </multiselect>
            </div>
        </div>
        <p class="form-control-static" v-else-if="items && displayMode === 'VIEW' && typeof items[0] === 'string'">\{{ clonedValue.value }}</p>
        <p class="form-control-static" v-else-if="items && displayMode === 'VIEW' && typeof items[0] === 'object'">\{{ displayFromObject }}</p>
    </div>
</template>

<script>
import Multiselect from 'vue-multiselect';
export default {
    name: 'SelectField',
    props: {
        options: {
            type: Object
        },
        value: {
            type: [String, Array, Object]
        },
        displayMode: {
            type: String
        },
        property: {
            type: Object
        },
        selectFrom: {
            type: [Object, Array]
        }
    },
    components: {
        Multiselect
    },
    data() {
        return {
            clonedValue: {},
            displayField: '',
            selected: []
        };
    },
    methods: {
        handler() {
            this.clonedValue.value = this.selected && this.selected['_id'] ? this.selected['_id'] : this.selected;
            this.validate();
            this.$emit('updateValue', this.clonedValue);
        },
        validate() {
            if (this.property) {
                if (this.property.required && !this.clonedValue.value && !this.multiple) {
                    this.clonedValue.$invalid = true;
                    this.clonedValue.$error = 'required';
                } else if (this.property.required && !this.clonedValue.value.length && this.multiple) {
                    this.clonedValue.$invalid = true;
                    this.clonedValue.$error = 'required';
                } else {
                    this.clonedValue.$invalid = false;
                    this.clonedValue.$error = null;
                }
            }
        }
    },
    computed: {
        items() {
            return this.selectFrom || (this.property ? this.property.enum : []);
        },
        displayFromObject() {
            let result;
            if(this.selectFrom.length) {
                result = this.selectFrom.find(item => item._id === this.clonedValue.value);
                return result ? result[this.displayField || 'Name'] : undefined;
            }
            return '';
        }
    },
    watch: {
        selectFrom: {
            handler: function(newVal) {
                if (newVal.length > 0 && typeof newVal[0] === 'object') {
                    this.selected = newVal.find(item => item._id === this.selected);
                }
            },
            deep: true
        }
    },
    created() {
        this.displayField = this.options.displayField;
        if (this.selectFrom && this.selectFrom.length && typeof this.selectFrom[0] === 'object') { // check if any external objects exist in the list  via select from
            if (this.value) {
                this.selected = this.selectFrom.find(item => item._id === this.value);
            } else if (this.property && this.property.value) {
                this.selected = this.selectFrom.find(item => item._id === this.property.value);
            } else {
                this.selected = undefined;
            }
        } else { // else treat the value as string
            if (this.value) {
                this.selected = this.value;
            } else if (this.property && this.property.value) {
                this.selected = this.property.value;
            } else {
                this.selected = undefined;
            }
        }
        this.handler();
    }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
