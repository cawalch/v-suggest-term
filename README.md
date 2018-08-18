# v-suggest-term

Suggests terms from an Object using a custom filter to return matches.

## Example Usage

```html
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
            >
            </textarea>
        </template>
    </vue-suggest-term>
```

```javascript
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
```

### fields 

Field Object or Array to match against.

### filter

Function to filter field matches

### limit

Max number of matching terms to display

### list-class

Optional CSS class to apply to child `ul`

### term-selected-class

Optional CSS class to apply to actively selected term

### selectTerm

Event emitted on term selection. Passes the selected term.

## Running Demo

```
yarn serve
```

## Known Issues

- CSS normalization needed for IE for proper positioning