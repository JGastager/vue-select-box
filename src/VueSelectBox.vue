<template>
    <div
        class="select-box"
        @click.self="toggleDropdown"
        tabindex="0"
        :style="dropdownStyle"
        @blur="checkBlur"
        @keyup.enter="openDropdown"
        @keyup.escape="closeDropdown"
        @keyup.up="navigateOptions('up')"
        @keyup.down="navigateOptions('down')"
    >
        <div class="selected" :style="selectedStyle" @click="toggleDropdown" aria-label="Toggle Dropdown">
            {{ findSelectedLabel() || placeholder }}
            <span class="caret" :style="caretStyle">
                <slot name="caret" v-if="hasCaretSlot"></slot>
                <template v-else>▼</template>
            </span>
            <Transition name="deselect">
                <span
                    v-if="clearable && selectedValue"
                    class="deselect"
                    aria-label="Clear Selection"
                    @click.stop="clearSelection"
                >
                    <slot name="deselect" v-if="hasDeselectSlot"></slot>
                    <template v-else>&#10005;</template>
                </span>
            </Transition>
        </div>
        <div ref="options" class="options" :style="dropdownOpen ? optionsStyle : 'height: 0;'">
            <div
                v-for="option in options"
                :key="getValue(option)"
                class="option"
                :class="{ active: getValue(option) === selectedValue }"
                data-value="option.value"
                :style="optionStyle"
                @click="selectOption(option)"
                @blur="checkBlur"
                @keyup.enter="selectOption(option)"
                tabindex="-1"
            >
                {{ getLabel(option) }}
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        modelValue: {
            type: [String, Number, Boolean, null],
            required: false,
            default: null,
        },
        options: {
            type: Array,
            required: true,
        },
        value: {
            type: String,
            required: false,
        },
        label: {
            type: String,
            required: false,
        },
        placeholder: {
            type: String,
            required: false,
        },
        height: {
            type: Number,
            required: false,
            default: 40,
        },
        maxHeight: {
            type: Number,
            required: false,
            default: 300,
        },
        clearable: {
            type: Boolean,
            required: false,
            default: true,
        },
    },
    emits: ["update:modelValue"],
    data() {
        return {
            dropdownOpen: false,
            selectedValue: null,
            focusedIndex: -1,
        };
    },
    computed: {
        hasCaretSlot() {
            return !!this.$slots.caret;
        },
        hasDeselectSlot() {
            return !!this.$slots.deselect;
        },
        dropdownStyle() {
            return {
                marginBottom: this.dropdownOpen
                    ? `-${Math.min(this.options.length * this.height, this.maxHeight)}px`
                    : "0",
                zIndex: this.dropdownOpen ? 1 : "auto",
            };
        },
        selectedStyle() {
            return {
                height: `${this.height}px`,
                lineHeight: `${this.height}px`,
            };
        },
        caretStyle() {
            return this.dropdownOpen ? { transform: "rotate(180deg)" } : {};
        },
        optionsStyle() {
            return {
                height: this.dropdownOpen ? `${Math.min(this.options.length * this.height, this.maxHeight)}px` : "0",
            };
        },
        optionStyle() {
            return {
                lineHeight: `${this.height}px`,
            };
        },
    },
    methods: {
        findSelectedLabel() {
            if (this.selectedValue) {
                const selectedOption = this.options.find((option) => this.getValue(option) === this.selectedValue);
                if (selectedOption) {
                    return this.getLabel(selectedOption);
                }
            }
            return null;
        },

        getNestedValue(obj, path) {
            const parts = path.replace(/\[(\w+)\]/g, ".$1").split("."); // Convert indices to properties
            return parts.reduce((o, key) => (o && key in o ? o[key] : null), obj);
        },

        getLabel(option) {
            if (this.label) {
                return this.getNestedValue(option, this.label);
            }
            return option.label || null;
        },

        getValue(option) {
            if (this.value) {
                return this.getNestedValue(option, this.value);
            }
            return option.value || null;
        },

        toggleDropdown() {
            this.dropdownOpen = !this.dropdownOpen;
            console.log("toggleDropdown", this.dropdownOpen);
        },
        closeDropdown() {
            this.dropdownOpen = false;
            console.log("closeDropdown", this.dropdownOpen);
        },
        openDropdown() {
            this.dropdownOpen = true;
            this.focusOption();
            console.log("openDropdown", this.dropdownOpen);
        },

        clearSelection() {
            this.selectedValue = null;
            this.$emit("update:modelValue", this.selectedValue);
            this.closeDropdown();
        },
        selectOption(option) {
            this.selectedValue = this.getValue(option);
            this.$emit("update:modelValue", this.selectedValue);
            this.closeDropdown();
        },

        checkBlur(event) {
            const target = event.relatedTarget;
            const optionsContainer = this.$refs.options;
            if (optionsContainer && optionsContainer.contains(target)) {
                // Clicked inside dropdown
                return;
            } else {
                // Clicked outside dropdown
                this.closeDropdown();
            }
        },

        checkClick(event) {
            const target = event.relatedTarget;
            const optionsContainer = this.$refs.options;
            if (optionsContainer && optionsContainer.contains(target)) {
                // Clicked inside dropdown
                console.log("checkClick inside");
                return;
            } else {
                // Clicked outside dropdown
                console.log("checkClick outside");
                this.openDropdown();
            }
        },

        navigateOptions(direction) {
            this.openDropdown();

            if (direction === "up") {
                this.focusedIndex = Math.max(0, this.focusedIndex - 1);
            } else if (direction === "down") {
                this.focusedIndex = Math.min(this.options.length - 1, this.focusedIndex + 1);
            }

            this.focusOption();
        },

        focusOption() {
            // Focus the option
            const optionsContainer = this.$refs.options;
            if (optionsContainer && optionsContainer.children.length > 0) {
                const focusedOption = optionsContainer.children[this.focusedIndex];
                if (focusedOption) {
                    focusedOption.focus();
                }
            }
        },
    },
};
</script>

<style>
.select-box .deselect-enter-active,
.select-box .deselect-leave-active {
    transition: opacity 0.1s ease;
}

.select-box .deselect-enter-from,
.select-box .deselect-leave-to {
    opacity: 0;
}

.select-box {
    box-sizing: border-box;
    background-color: white;
    border: 1px solid #cccccc;
    cursor: pointer;
    text-align: left;
    border-radius: 5px;
    transition: margin-bottom 0.3s ease;
    overflow: hidden;
}

.select-box:focus,
.select-box:focus-within {
    /* outline: none; */
    outline-color: #eeeeee;
    outline-width: 3px;
    outline-offset: 0;
    outline-style: solid;
}

.select-box .selected {
    /* Style as needed */
    box-sizing: border-box;
    padding: 0 1rem;
}

.select-box .options {
    /* Style for dropdown options */
    box-sizing: border-box;
    transition: height 0.3s ease;
    overflow: auto;
}

.select-box .option {
    /* Style individual options */
    box-sizing: border-box;
    padding: 0 1rem;
    transition: background-color 0.2s ease;
}

.select-box .option.active {
    background-color: #f8f8f8;
}

.select-box .option:hover,
.select-box .option:focus {
    background-color: #eeeeee;
    outline: none;
}

.select-box .deselect {
    /* Style the deselect button */
    box-sizing: border-box;
    float: right;
    margin-left: 10px;
    cursor: pointer;
}

.select-box .caret {
    /* Style the caret */
    box-sizing: border-box;
    float: right;
    margin-left: 10px;
    cursor: pointer;
    transition: transform 0.3s ease;
}
</style>
