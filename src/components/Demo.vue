<template type="html">
    <div class="container">
        <vue-suggest-term 
        :fields="mappings"
        :filter="filter"
        :limit="limit"
        :list-class="'app-ul'"
        :term-selected-class="'selected'"
        @selectTerm="updateQuery"
        >
        <template slot="input">
            <textarea
                spellcheck="false"
                v-model="query"    
            ></textarea>
        </template>
        </vue-suggest-term>
    </div>
</template>

<script>
import VueSuggestTerm from './VueSuggestTerm.vue'
export default {
    data() {
        return {
            query: '',
            mappings: [
                { name: 'foobar' },
                { name: 'foobell' },
                { name: 'foobull' },
                { name: '$someVal'},
                { name: 'MATCH('},
            ],
            limit: 5,
            filter(val, mappings, limit) {
                const ret = []
                for (let i = 0; i < mappings.length && i < limit; i += 1) {
                    const entry = mappings[i]
                    if (entry.name.startsWith(val) ||
                    entry.name.endsWith(val)) {
                        ret.push(entry)
                    }
                }
                return ret
            },
        }
    },
    methods: {
        updateQuery(v) {
            this.query = v
        },
    },
    components: {
        VueSuggestTerm,
    }
}
</script>

<style>
.app-ul li {
    cursor: pointer;
    font-family: monospace;
    padding: 3px;
    font-size: 12px;
    font-weight: 700;
}

.app-ul {
    background-color: white;
    border: 1px solid #dfdfdf;
    margin-left: 5px;
}

.app-ul li.selected {
    background: #dedede;
}
</style>