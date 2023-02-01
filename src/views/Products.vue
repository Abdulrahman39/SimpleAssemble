<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount } from 'vue';
import ProductService from '@/service/ProductService';
import { useToast } from 'primevue/usetoast';
import { useLayout } from '@/layout/composables/layout';
import axios from 'axios';

const toast = useToast();
const { contextPath } = useLayout();
const imageF = ref([])
const products = ref([]);
const productDialog = ref(false);
const deleteProductDialog = ref(false);
const deleteProductsDialog = ref(false);
const product = ref({});
const selectedProducts = ref(null);
const dt = ref(null);
const filters = ref({});
const submitted = ref(false);
const statuses = ref([
    { label: 'INSTOCK', value: 'instock' },
    { label: 'LOWSTOCK', value: 'lowstock' },
    { label: 'OUTOFSTOCK', value: 'outofstock' }
]);


const category = ref([])

const productService = new ProductService();
const uid = window.localStorage.getItem("uid")
onBeforeMount(() => {
    initFilters();
});
onMounted(async () => {
    await axios.get(`https://simple-assemble-af150-default-rtdb.firebaseio.com/products/${uid}.json`).then((data) => {
        const allP = []
        const res = data.data
        for (const key in res) {
            allP.push(
                {
                    id: key,
                    name: res[key].name,
                    description: res[key].description,
                    images: res[key].images,
                    videoLink: res[key].videoLink,
                    category: res[key].category
                }
            )

        }
        products.value = allP
        console.log(allP, product.value)


    });


});

async function Data() {
    await axios.get(`https://simple-assemble-af150-default-rtdb.firebaseio.com/products/${uid}.json`).then((data) => {
        const allP = []
        const res = data.data
        for (const key in res) {
            allP.push(
                {
                    id: key,
                    name: res[key].name,
                    description: res[key].description,
                    images: res[key].images,
                    videoLink: res[key].videoLink,
                    category: res[key].category
                }
            )

        }
        products.value = allP
        console.log(allP, product.value)


    });
}
const formatCurrency = (value) => {
    return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
};
const onSelectedFiles = (event) => {
    imageF.value = event.files;
};
const openNew = () => {
    product.value = {};
    submitted.value = false;
    productDialog.value = true;
};

const hideDialog = () => {
    productDialog.value = false;
    submitted.value = false;
};

const saveProduct = async () => {
    submitted.value = true;
    if (product.value.name && product.value.name.trim()) {
        {
            product.value.category = category.value;
            product.value.image = imageF.value;
            const headers = {
                'Authorization': window.localStorage.getItem("Token"),
                'Content-Type': 'application/json',
            };
            const data = {
                name: product.value.name,
                description: product.value.description,
                category: product.value.category,
                images: imageF.value,
                videoLink: product.value.videoLink,

            }
            console.log(data, imageF.value[0].name, "VVVVVVVVVVVVVVVVVVVVVVVVVV")
            await axios.post(`https://simple-assemble-af150-default-rtdb.firebaseio.com/products/${uid}.json`, data, { headers }).then((data) => {
                product.value.id = data.data.name;
                products.value.push(product.value);
                Data()
                toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Created', life: 3000 });
            })

        }
        productDialog.value = false;
        product.value = {};
    }
};

const editProduct = (editProduct) => {
    product.value = { ...editProduct };
    console.log(product);
    productDialog.value = true;
};

const confirmDeleteProduct = (editProduct) => {
    product.value = editProduct;
    deleteProductDialog.value = true;
};

const deleteProduct = () => {
    products.value = products.value.filter((val) => val.id !== product.value.id);
    deleteProductDialog.value = false;
    product.value = {};
    toast.add({ severity: 'success', summary: 'Successful', detail: 'Product Deleted', life: 3000 });
};

const findIndexById = (id) => {
    let index = -1;
    for (let i = 0; i < products.value.length; i++) {
        if (products.value[i].id === id) {
            index = i;
            break;
        }
    }
    return index;
};

const createId = () => {
    let id = '';
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    for (let i = 0; i < 5; i++) {
        id += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return id;
};

const exportCSV = () => {
    dt.value.exportCSV();
};

const confirmDeleteSelected = () => {
    deleteProductsDialog.value = true;
};
const deleteSelectedProducts = () => {
    products.value = products.value.filter((val) => !selectedProducts.value.includes(val));
    deleteProductsDialog.value = false;
    selectedProducts.value = null;
    toast.add({ severity: 'success', summary: 'Successful', detail: 'Products Deleted', life: 3000 });
};

const initFilters = () => {
    filters.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS }
    };
};
</script>

<template>
    <div class="grid">
        <div class="col-12">
            <div class="card">
                <Toast />
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <div class="my-2">
                            <Button label="New" icon="pi pi-plus" class="p-button-success mr-2" @click="openNew" />
                            <Button label="Delete" icon="pi pi-trash" class="p-button-danger"
                                @click="confirmDeleteSelected"
                                :disabled="!selectedProducts || !selectedProducts.length" />
                        </div>
                    </template>


                </Toolbar>

                <DataTable ref="dt" :value="products" v-model:selection="selectedProducts" dataKey="id"
                    :paginator="true" :rows="10" :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Showing {first} to {last} of {totalRecords} products"
                    responsiveLayout="scroll">
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Manage Products</h5>
                            <span class="block mt-2 md:mt-0 p-input-icon-left">
                                <i class="pi pi-search" />
                                <InputText v-model="filters['global'].value" placeholder="Search..." />
                            </span>
                        </div>
                    </template>

                    <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>

                    <Column field="id" header="ID" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">ID</span>
                            {{ slotProps.data.id }}
                        </template>
                    </Column>
                    <Column field="name" header="Name" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Name</span>
                            {{ slotProps.data.name }}
                        </template>
                    </Column>
                    <Column field="description" header="Description" :sortable="true"
                        headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Name</span>
                            {{ slotProps.data.description }}
                        </template>
                    </Column>
                    <Column header="image" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Image</span>
                            <Galleria :value="slotProps.images" :responsiveOptions="responsiveOptions" :numVisible="5"
                                :circular="true" containerStyle="max-width: 140px" :showItemNavigators="true">
                                <template #item="slotProps">
                                    <img :src="slotProps.item.itemImageSrc" :alt="slotProps.item.alt"
                                        style="width: 100%; display: block;" />
                                </template>
                                <template #thumbnail="slotProps">
                                    <img :src="slotProps.item.thumbnailImageSrc" :alt="slotProps.item.alt"
                                        style="display: block;" />
                                </template>
                            </Galleria>
                        </template>
                    </Column>
                    <Column header="video" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Video</span>
                            {{ slotProps.data.videoLink }}
                        </template>
                    </Column>

                    <Column field="category" header="Category" :sortable="true"
                        headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Category</span>
                            {{ slotProps.data.category }}
                        </template>
                    </Column>


                    <!-- <Column field="inventoryStatus" header="Status" :sortable="true"
                        headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Status</span>
                            <span
                                :class="'product-badge status-' + (slotProps.data.inventoryStatus ? slotProps.data.inventoryStatus.toLowerCase() : '')">{{
                                slotProps.data.inventoryStatus }}</span>
                        </template>
                    </Column> -->
                    <Column headerStyle="min-width:10rem;">
                        <template #body="slotProps">
                            <Button icon="pi pi-pencil" class="p-button-rounded p-button-success mr-2"
                                @click="editProduct(slotProps.data)" />
                            <Button icon="pi pi-trash" class="p-button-rounded p-button-warning mt-2"
                                @click="confirmDeleteProduct(slotProps.data)" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="productDialog" :style="{ width: '450px' }" header="Product Details"
                    :modal="true" class="p-fluid">

                    <div class="field">
                        <label for="name">Name</label>
                        <InputText id="name" v-model.trim="product.name" required="true" autofocus
                            :class="{ 'p-invalid': submitted && !product.name }" />
                        <small class="p-invalid" v-if="submitted && !product.name">Name is required.</small>
                    </div>
                    <div class="field">
                        <label for="description">Description</label>
                        <Textarea id="description" v-model="product.description" required="true" rows="3" cols="20" />
                        <!-- <small class="p-invalid" v-if="submitted && !product.description">Name is required.</small> -->
                    </div>



                    <div class="field">
                        <label class="mb-3">Category</label>
                        <div class="formgrid grid col ">
                            <div class="field-checkbox col">
                                <Checkbox inputId="city2" name="city" value="House" v-model="category" />
                                <label for="city2">House</label>
                            </div>
                            <div class="field-checkbox col">
                                <Checkbox inputId="city3" name="city" value="Furniture" v-model="category" />
                                <label for="city3">Furniture</label>
                            </div>
                            <div class="field-checkbox col">
                                <Checkbox inputId="city4" name="city" value="Electronics" v-model="category" />
                                <label for="city4">Electronics</label>
                            </div>
                        </div>
                    </div>

                    <div class="field ">
                        <label for="price" :class="{ 'p-invalid': submitted && !product.price }">Images</label>
                        <FileUpload name="imageF" :multiple="true" :maxFileSize="1000000" :fileLimit="3"
                            @select="onSelectedFiles" />
                        <small class="p-invalid" v-if="submitted && !product.images">Images are required.</small>
                    </div>
                    <div class="field">
                        <label for="video">video Link</label>
                        <InputText id="name" v-model.trim="product.videoLink" />
                    </div>
                    <template #footer>
                        <Button label="Cancel" icon="pi pi-times" class="p-button-text" @click="hideDialog" />
                        <Button label="Save" icon="pi pi-check" class="p-button-text" @click="saveProduct" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteProductDialog" :style="{ width: '450px' }" header="Confirm"
                    :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="product">Are you sure you want to delete <b>{{ product.name }}</b>?</span>
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" class="p-button-text"
                            @click="deleteProductDialog = false" />
                        <Button label="Yes" icon="pi pi-check" class="p-button-text" @click="deleteProduct" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteProductsDialog" :style="{ width: '450px' }" header="Confirm"
                    :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="product">Are you sure you want to delete the selected products?</span>
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" class="p-button-text"
                            @click="deleteProductsDialog = false" />
                        <Button label="Yes" icon="pi pi-check" class="p-button-text" @click="deleteSelectedProducts" />
                    </template>
                </Dialog>
            </div>
        </div>
    </div>
</template>

<style scoped lang="scss">
@import '@/assets/demo/styles/badges.scss';
</style>
