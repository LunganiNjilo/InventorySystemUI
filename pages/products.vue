<template>
    <NavBar />

    <v-container>
        <v-row>
            <PageHeader :title="pageTitle" :text="pageDescription" />
        </v-row>
    </v-container>

    <DataFetcher :uri="apiBaseUri">
        <template #default="{ data, create, edit, deleteItem, refresh }">
            <!-- Force table update when data changes -->
            <TableComponent
                :key="JSON.stringify(data?.result)"
                :title="pageTitle"
                :entityName="entityName"
                :headers="headers"
                :data="data"
                :entityKeys="entityKeys"
                :create="create"
                :edit="edit"
                :deleteItem="deleteItem"
                :refresh="refresh"
                :openAddDialog="openAddDialog"
                :openEditDialog="openEditDialog"
                :openDetailsDialog="openDetailsDialog"
                :deleteEntity="(id) => deleteEntity(id, deleteItem, refresh)"
            />

            <!-- PRODUCT DIALOG -->
            <v-dialog v-model="dialog" max-width="600px">
                <v-card>
                    <v-card-title>
                        <span class="headline">{{ dialogTitle }}</span>
                    </v-card-title>

                    <v-card-text>
                        <v-container>
                            <v-row>
                                <v-col
                                    cols="12"
                                    v-for="(field, index) in entityFields"
                                    :key="index"
                                >
                                    <!-- TEXT INPUTS -->
                                    <v-text-field
                                        v-if="field.key !== 'fkProductCategory'"
                                        v-model="editedEntity[field.key]"
                                        :label="field.label"
                                        :rules="field.rules"
                                        :placeholder="field.placeholder"
                                        :readonly="isDetailsDialog"
                                    />

                                    <!-- CATEGORY DROPDOWN -->
                                    <v-select
                                        v-else
                                        v-model="editedEntity.fkProductCategory"
                                        :items="fkOptions.fkProductCategory"
                                        item-title="productCategoryName"
                                        item-value="id"
                                        label="Category"
                                        :rules="[v => !!v || 'Category is required']"
                                        required
                                        :readonly="isDetailsDialog"
                                    />
                                </v-col>
                            </v-row>
                        </v-container>
                    </v-card-text>

                    <v-card-actions>
                        <v-btn color="blue darken-1" text @click="close">Close</v-btn>

                        <v-btn
                            color="#4caf50"
                            text
                            v-if="!isDetailsDialog"
                            @click="save(create, edit, refresh)"
                        >
                            Save
                        </v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </template>
    </DataFetcher>

    <PageFooter />

    <!-- ⭐ SNACKBAR FOR ERROR MESSAGES -->
    <v-snackbar
        v-model="snackbar"
        timeout="4000"
        color="red"
        elevation="24"
    >
        {{ snackbarMessage }}
    </v-snackbar>
</template>

<script setup>
import { ref, onMounted } from "vue";

const pageTitle = "Products";
const pageDescription = "Manage your products here.";
const entityName = "Product";

const headers = [
    "Product Name",
    "Description",
    "Sale Price",
    "Cost Price",
    "Category"
];

const entityKeys = [
    "productName",
    "productDescription",
    "sellPrice",
    "costPrice",
    "productCategoryName",
    "fkProductCategory"
];

const apiBaseUri = "http://localhost:8080/api/Products";

const dialog = ref(false);
const dialogTitle = ref("");
const isDetailsDialog = ref(false);
const currentEntityId = ref(null);

const editedEntity = ref({
    productName: "",
    productDescription: "",
    sellPrice: 0,
    costPrice: 0,
    fkProductCategory: null
});

const fkOptions = ref({
    fkProductCategory: []
});

// ⭐ SNACKBAR PROPERTIES
const snackbar = ref(false);
const snackbarMessage = ref("");

// Load category options (with auth header)
const fetchCategoryOptions = async () => {
    try {
        const token = localStorage.getItem("token");

        const response = await $fetch(
            "http://localhost:8080/api/ProductCategories",
            {
                headers: token
                    ? { Authorization: `Bearer ${token}` }
                    : {}
            }
        );

        const items = response?.result ?? [];
        fkOptions.value.fkProductCategory = items.map(c => ({
            id: c.id,
            productCategoryName: c.productCategoryName
        }));
    } catch (err) {
        console.error("Failed loading categories:", err);
    }
};

onMounted(fetchCategoryOptions);

const entityFields = [
    { key: "productName", label: "Product Name", rules: [v => !!v || "Required"] },
    { key: "productDescription", label: "Product Description" },
    { key: "sellPrice", label: "Sale Price", rules: [v => !!v || "Required"] },
    { key: "costPrice", label: "Cost Price", rules: [v => !!v || "Required"] },
    { key: "fkProductCategory", label: "Category" }
];

const openAddDialog = () => {
    resetForm();
    dialogTitle.value = `Add New ${entityName}`;
    dialog.value = true;
};

const openEditDialog = (id, entity) => {
    resetForm();
    currentEntityId.value = id;

    editedEntity.value = {
        productName: entity.productName,
        productDescription: entity.productDescription,
        sellPrice: entity.sellPrice,
        costPrice: entity.costPrice,
        fkProductCategory: entity.fkProductCategory
    };

    dialogTitle.value = `Edit ${entityName}`;
    dialog.value = true;
};

const openDetailsDialog = (id, entity) => {
    resetForm();
    currentEntityId.value = id;

    editedEntity.value = { ...entity };

    isDetailsDialog.value = true;
    dialogTitle.value = `Details of ${entityName}`;
    dialog.value = true;
};

// ⭐ UPDATED DELETE WITH ERROR HANDLING
const deleteEntity = async (id, deleteItem, refresh) => {
    try {
        await deleteItem(id);   // backend attempt
        await refresh();        // UI refresh
    } catch (err) {
        const message =
            err?.response?._data?.error ||
            err?.data?.error ||
            err?.message ||
            "Unable to delete product.";

        snackbarMessage.value = message;
        snackbar.value = true;
    }
};

const close = () => {
    dialog.value = false;
    resetForm();
};

const save = async (createFunction, editFunction, refresh) => {
    const cleanEntity = JSON.parse(JSON.stringify(editedEntity.value));

    if (currentEntityId.value) {
        await editFunction(currentEntityId.value, cleanEntity);
    } else {
        await createFunction(cleanEntity);
    }

    dialog.value = false;
    resetForm();
    await refresh();
};

const resetForm = () => {
    isDetailsDialog.value = false;
    currentEntityId.value = null;

    editedEntity.value = {
        productName: "",
        productDescription: "",
        sellPrice: 0,
        costPrice: 0,
        fkProductCategory: null
    };
};
</script>

<style scoped>
/* Original styling preserved */
</style>
