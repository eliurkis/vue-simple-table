<template>
    <div class="table-responsive simple-table">
        <table class="table table-striped table-hover">
            <thead>
            <tr>
                <th v-for="column in columns"
                    :class="{'simple-table__sorteable': isSorteable(column)}"
                    @click="setOrderBy(column)">
                    <span class="simple-table__heading">
                        <slot :name="`header-${column}`" v-bind:column="column">
                            {{ get(options.headings, column, prepareHeading(column)) }}
                        </slot>
                    </span>
                    <span v-if="isSorteable(column)"
                          class="simple-table__sort-icon float-right fal"
                          :class="defineSortClass(column)">
                    </span>
                </th>
            </tr>
            </thead>
            <tbody>
            <tr v-show="showLoadingProgress">
                <td :colspan="columns.length">
                    <spinner></spinner>
                </td>
            </tr>
            <tr v-for="row in rows" v-show="!showLoadingProgress">
                <td v-for="column in columns">
                    <slot :name="column" v-bind:row="row">
                        {{ get(row, column) }}
                    </slot>
                </td>
            </tr>
            </tbody>
            <tfoot v-show="!showLoadingProgress">
            <tr>
                <td :colspan="columns.length" class="simple-table__pagination-footer">
                    <div class="simple-table__pagination-footer__label">
                        <slot name="pagination-label" v-bind:data="{pagination, total, curPage}">
                            Showing {{ pagination.from }} to {{ pagination.to }} of {{ total }} records
                        </slot>
                    </div>
                    <div>
                        <slot name="pagination-paginate" v-bind:data="{pagination, total, curPage}">
                            <paginate v-model="curPage" :total-count="total" :per-page="localPerPage"></paginate>
                        </slot>
                    </div>
                </td>
            </tr>
            </tfoot>
        </table>
    </div>
</template>

<script>
    import axios from 'axios'
    import get from 'lodash/get'
    import Paginate from './av-paginate'
    import Spinner from 'vue-simple-spinner'

    export default {
        name: "SimpleTable",

        props: {
            columns: {
                type: Array,
                default: () => ([])
            },
            data: {
                type: Array,
                default: () => ([])
            },
            options: {
                type: Object,
                default: () => ({})
            },
            url: {
                type: String,
                default: null
            },
            perPage: {
                type: Number,
                default: 15
            },
            currentPage: {
                type: Number,
                default: 1
            },
            params: {
                type: Object,
                default: () => ({})
            },
            showLoading: {
                type: Boolean,
                default: true
            },
            orderBy: {
                type: String,
                default: null
            },
            sortBy: {
                type: String,
                default: 'asc'
            }
        },

        data() {
            return {
                isLoading: true,
                curPage: this.currentPage,
                localPerPage: this.perPage,
                localData: this.data,
                total: this.data.length,
                pagination: {
                    from: 0,
                    to: 0,
                },
                localOrderBy: this.orderBy,
                localSortBy: this.sortBy.toLowerCase()
            }
        },

        watch: {
            curPage() {
                if (this.isRemote) {
                    this.fetchRows()
                }
            },
            localSortBy() {
                if (this.isRemote) {
                    this.fetchRows()
                }
            },
            localOrderBy() {
                if (this.isRemote) {
                    this.fetchRows()
                }
            }
        },

        computed: {
            isRemote() {
                return this.url !== null
            },

            rows() {
                // Remote Data
                if (this.isRemote) {
                    return this.localData
                }

                // Local Data
                const offset = (this.curPage - 1) * this.perPage
                return this.data.slice(offset, offset + this.perPage)
            },

            showLoadingProgress() {
                return this.isLoading && this.showLoading
            }
        },

        methods: {
            prepareHeading(str) {
                return (str + '')
                    .replace('.', ' ')
                    .replace(/^(.)|\s+(.)/g, function ($1) {
                        return $1.toUpperCase()
                    })
            },

            get(object, keys, defaultVal = null) {
                return get(object, keys, defaultVal)
            },

            fetchRows() {
                const component = this
                const queryString = Object.keys(this.params).map((key) => {
                    return encodeURIComponent(key) + '=' + encodeURIComponent(this.params[key])
                }).join('&')
                const orderByQuery = this.orderBy ? `&order_by=${this.localOrderBy}&sort_by=${this.localSortBy}` : '';
                const url = `${this.url}?page=${this.curPage}&per_page=${this.perPage}${orderByQuery}${queryString ? '&' + queryString : ''}`

                this.isLoading = true
                axios.get(url)
                    .then(({ data }) => {
                        component.localData = data.data
                        component.total = data.total
                        component.curPage = data.current_page
                        component.pagination = {
                            from: data.from,
                            to: data.to,
                        }
                        component.isLoading = false
                    })
                    .catch(() => {
                        component.isLoading = false
                    })
            },

            setOrderBy(column) {
                if (!this.isSorteable(column)) {
                    return
                }

                this.localSortBy = this.isOrderBy(column) && this.localSortBy.toLowerCase() === 'asc' ? 'desc' : 'asc'
                this.localOrderBy = column
            },

            isOrderBy(column) {
                return this.localOrderBy === column
            },

            isSorteable(column) {
                return (get(this.options, 'sortable') || []).includes(column)
            },

            defineSortClass(column) {
                if (!this.isOrderBy(column)) {
                    return 'fa-sort'
                }

                return this.localSortBy.toLowerCase() === 'desc'
                    ? 'fa-sort-alpha-desc'
                    : 'fa-sort-alpha-asc'
            }
        },

        mounted() {
            if (this.isRemote) {
                this.fetchRows()
            }
        },

        components: {
            Paginate,
            Spinner
        }
    }
</script>

<style>
    .simple-table__sorteable {
        cursor: pointer;
    }

    .simple-table__sort-icon {
        padding-top: 3px;
        width: 10px;
        text-align: center;
    }

    .simple-table__pagination-footer {
        padding: 10px 0;
    }

    .simple-table__pagination-footer div {
        width: 50%;
        float: left;
    }

    .simple-table__pagination-footer__label {
        font-size: 13px;
        padding-top: 5px;
    }

    .simple-table .pagination {
        float: right !important;
    }
</style>
