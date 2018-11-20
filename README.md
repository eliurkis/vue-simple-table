# Vue Simple Table
Vue Tables made simple!

## Example

```html
<vue-simple-table :data="rows"
          :columns="columns"
          :options="options"
          :per-page="14"
          :current-page="1"
          :params="{'anotherParam': 1}"
          :show-loading="false"
          order-by="number"
          url="/api/customers">
    <template slot="header-number" slot-scope="{column}">
        Number Col
    </template>
    <template slot="number" slot-scope="{row}">
        #{{row.number}}
    </template>
    <actions slot="actions" slot-scope="{row}" style="text-align: center">
        <a class="dropdown-item" @click="edit(row.id)"><i class="fal fa-edit"></i> Edit</a>
        <a class="dropdown-item" @click="remove(row.id)"><i class="fal fa-remove"></i> Delete</a>
    </actions>
</vue-simple-table>
```

```js
new Vue({
  el: "#app",
  data: {
    columns: [
        'number',
        'date',
        'customer.name',
        'reference',
        'priority.description',
        'quoted_by.profile.full_name',
        'actions'
    ],
    rows: [],
    options: {
        sortable: [
            'number',
            'date',
            'customer.name',
            'reference',
            'priority.description',
            'quoted_by.profile.full_name'
        ],
        headings: {
            'customer.name': 'Customer Name',
            'priority.description': 'Priority',
            'quoted_by.profile.full_name': 'Quoted By'
        },
        columnsClasses: {
            'actions': 'text-center'
        }
    }
}
})
```
