<template>
    <div class="container">
        <div class="row">
            <div v-for="(category, index) in categories" :key="index" class="col-12">
                <!-- LEVEL 1 -->
                <div class="p-1">
                    <font-awesome-icon
                        icon="caret-right"
                        :transform="{ rotate: category.rotate || 0 }"
                        :class="{ transparent: !category.sub_category.length }"
                    />
                    <span
                        class="pl-1 pointer"
                        @click="toggle(index)"
                    >
                        {{ category.category_name }}
                    </span>
                </div>
                <div class="pl-4" v-show="category.rotate">
                    <div v-for="(subcategory, index2) in category.sub_category" :key="index2">
                        <!-- LEVEL 2 -->
                        <div class="p-1">
                            <font-awesome-icon
                                icon="caret-right"
                                :transform="{ rotate: subcategory.rotate || 0 }"
                                :class="{ transparent: !subcategory.sub_category.length }"
                            />
                            <span
                                class="pl-1 pointer"
                                @click="toggle(index, index2)"
                            >
                                {{ subcategory.category_name }}
                            </span>
                        </div>
                        <div class="pl-4" v-show="subcategory.rotate">
                            <div v-for="(subsubcategory, index3) in subcategory.sub_category" :key="index3">
                                <!-- LEVEL 3 -->
                                <div class="p-1">
                                    <font-awesome-icon
                                        icon="caret-right"
                                        class="transparent"
                                    />
                                    <span
                                        class="pl-1 pointer"
                                    >
                                        {{ subsubcategory.category_name }}
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="col-md-12">
                <div class="p-2">
                    <base-button type="primary">
                        <font-awesome-icon icon="plus" class="mr-2"/>
                        Add Category
                    </base-button>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    name: 'manage-category',
    data: () => ({
        categories: [],
    }),
    methods: {
        getCategories() {
            // Get list of all categories and sub categories
            this.$axios({
                method: 'get',
                url: '/inventory/categories/all',
            }).then((response) => {
                // assign to this.categories.
                this.categories = response.data.data.categories;
            });
        },
        toggle(index, index2) {
            if(index2 === undefined){
                if(this.categories[index].rotate) {
                    this.$set(this.categories[index], 'rotate', 0);
                } else {
                    this.$set(this.categories[index], 'rotate', 90);
                }
            } else {
                if(this.categories[index].sub_category[index2].rotate) {
                    this.$set(this.categories[index].sub_category[index2], 'rotate', 0);
                } else {
                    this.$set(this.categories[index].sub_category[index2], 'rotate', 90);
                }
            }
        },
    },
    mounted() {
        this.getCategories();
    },
}
</script>
<style scoped>
.pointer {
    color: #5e72e4;
    font-weight: bold;
}

.pointer:hover {
    color: #233dd2;
    cursor: pointer;
}

.transparent {
    opacity: 0;
}
</style>
