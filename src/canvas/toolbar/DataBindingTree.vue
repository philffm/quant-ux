
<template>
     <div class="MatcDataSettings">
         <div v-if="model" class="MatcDialogTable">
             <table>
                  <tr >
                    <td class="MatcDataBindingCheckCntr">
                        <span class="mdi mdi-database-plus"></span>
                    </td>
                     <td class="MatcDataBindingNameCntr">
                       <Combo
                            :fireOnBlur="true"
                            :top="false"
                            placeholder="Create new variable"
                            :inline="true"
                            :hints="hints"
                            ref="combo"
                            @focus="hasNewTypeSelector = true"
                            @change="onNewVariable"
					        :formControl="true"/>
                    </td>
                     <td>
                         <span class="MatcButton" v-if="hasNewTypeSelector">Add</span>
                    </td>
                 </tr>
             </table>

             {{dataModel}}
         </div>
	</div>
</template>
<style>

</style>
<script>
import DojoWidget from 'dojo/DojoWidget'
import lang from 'dojo/_base/lang'
import Util from 'core/Util'
import Logger from 'common/Logger'
// import DropDownButton from 'page/DropDownButton'
//import CheckBox from 'common/CheckBox'
import Input from 'common/Input'
//import SegmentButton from 'page/SegmentButton'

export default {
    name: 'DataBinding',
    mixins:[DojoWidget, Util],
    props:["app", "value", "canChangeVars"],
    data: function () {
        return {
            model: null,
            widget: null,
            hasNewTypeSelector: false,
            newType: "default",
            variables: [],
            databinding: {},
            dataModel: {}
        }
    },
    components: {
        //'CheckBox': CheckBox,
        'Combo': Input,
        //'SegmentButton': SegmentButton
        // 'DropDownButton': DropDownButton
    },
    computed: {
        dataTypes () {
            return ["Number", "String", "Boolean", "Object", "Array"].map(a => {
                return {
                    label: a,
                    value: a
                }
            })
        },
        variableKeys () {
            if (this.widget.type === 'Repeater') {
                return [
                    {
                        label: "In",
                        value: "default"
                    },
                    {
                        label: "Out",
                        value: "output"
                    }
                ]
            } else {
                return [
                    {
                        label: "In & out",
                        value: "default"
                    }
                ]
            }

        },
        hints () {
           	var hints = this.getHintsAppVariables();
			hints = hints.map(h => {
				return {
					label: h,
					value: h
				}
            })
            return hints
        },
        selectedVaribales () {
            let values2Keys = {}
            for (let key in this.databinding) {
                let value = this.databinding[key]
                values2Keys[value] = key
            }
			let result = this.variables.map(v => {
                return {
                    name: v,
                    selected: values2Keys[v] !== null && values2Keys[v] !== undefined,
                    defaultValue: '',
                    dataType: "Object",
                    type: values2Keys[v]
                }
            })
            return result
        }
    },
    methods: {
        onNewVariable (v) {
            this.$refs.combo.clear()
            if (this.variables.indexOf(v) < 0) {
                this.variables.unshift(v)
            }
            this.onSelectVariable(v, this.newType)
            this.hasNewTypeSelector = false
            this.onChange()
        },
        onCheckBox (selected, name, key) {
            if (selected) {
                this.onSelectVariable(name)
            } else {
                this.$delete(this.databinding, key)
                this.onChange()
            }
        },
        onSelectVariable (v, key = "default") {
            console.debug('onSelectVariable()', v, key)
            this.$set(this.databinding, key, v)
            this.onChange()
        },
        setNewType (newKey) {
            this.newType = newKey
        },
        setType (newKey, name) {
            let oldKey = null
            for (oldKey in this.databinding) {
                let value = this.databinding[oldKey]
                if (value === name) {
                    break
                }
            }
            if (oldKey) {
                this.$delete(this.databinding, oldKey)
                this.onSelectVariable(name, newKey)
            }
            this.onChange()
        },
        getValue () {
            return this.databinding
        },
        onChange () {
            this.emit('change', this.databinding)
        },
        setModel (v) {
            this.model = v
            if (v.dataModel) {
                this.dataModel = lang.clone(v.dataModel)
            }
        },
        setWidget (v) {
            this.widget = v
            if (this.widget.props && this.widget.props.databinding) {
                this.databinding = this.widget.props.databinding
            }
            this.initVariables()
        },
        initVariables () {
            var variables = this.getAllAppVariables();
            if (this.variables.length === 0) {
                variables.sort((a, b) => {
                    return a.localeCompare(b)
                })
            }
            this.variables = variables
            if (this.variables.length === 0) {
                setTimeout(() => {
                    if (this.$refs.combo) {
                        this.$refs.combo.focus()
                    }
                }, 200)
            }
        }
    },
    watch: {
        value (v) {
            this.setWidget(v)
        }
    },
    mounted () {
        this.logger = new Logger("DataSetting")
        if (this.app) {
            this.setModel(this.app)
        }
        if (this.value) {
            this.setWidget(this.value)
        }
    }
}
</script>