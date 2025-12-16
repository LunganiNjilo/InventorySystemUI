<template>
    <NavBar />

    <v-container>
        <v-row>
            <PageHeader title="Inventory" text="Manage your inventory here." />
        </v-row>
    </v-container>

    <DataFetcher :uri="apiBaseUri">
        <template #default="{ data, create, edit, deleteItem, refresh }">

            <TableComponent
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
                :deleteEntity="deleteEntity"
            />

            <v-dialog v-model="dialog" max-width="600px">
                <v-card>
                    <v-card-title>
                        <span class="headline">{{ dialogTitle }}</span>
                    </v-card-title>

                    <v-card-text>
                        <v-container>
                            <v-row>
                                <!-- PRODUCT DROPDOWN (NEW FIELD) -->
                                <v-col cols="12">
                                    <v-select
                                        v-model="editedEntity.fkProductId"
                                        :items="productOptions"
                                        item-title="productName"
                                        item-value="id"
                                        label="Product"
                                        :rules="[v => !!v || 'Product is required']"
                                        required
                                        :disabled="isDetailsDialog"
                                    />
                                </v-col>

                                <!-- OTHER FIELDS -->
                                <v-col cols="12" v-for="(field, index) in entityFields" :key="index">
                                    <v-text-field
                                        v-model.number="editedEntity[field.key]"
                                        :label="field.label"
                                        :rules="field.rules"
                                        type="number"
                                        :readonly="isDetailsDialog"
                                    />
                                </v-col>
                            </v-row>
                        </v-container>
                    </v-card-text>

                    <v-card-actions>
                        <v-btn text color="red" @click="close">Close</v-btn>
                        <v-btn text color="green" @click="() => save(create, edit, refresh)" v-if="!isDetailsDialog">Save</v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </template>
    </DataFetcher>

    <PageFooter />
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useFetch } from "#app";

const pageTitle = "Inventory";
const pageDescription = "Administer your inventory here.";
const entityName = "Inventory";

const headers = ["Product", "Quantity In Stock", "Min Stock Level", "Max Stock Level"];

const entityKeys = ["productName", "quantityInStock", "minStockLevel", "maxStockLevel"];

const apiBaseUri = "http://localhost:8080/api/Inventory";

// NEW: product dropdown options
const productOptions = ref([]);

// Fetch product list to select from
const loadProducts = async () => {
    const { data, error } = await useFetch("http://localhost:8080/api/Products");
    if (!error.value) {
        productOptions.value = data.value?.result ?? [];
    }
};

onMounted(loadProducts);

// Fields except product
const entityFields = [
    { key: "quantityInStock", label: "Quantity In Stock", rules: [v => v >= 0 || "Invalid value"] },
    { key: "minStockLevel", label: "Min Stock Level", rules: [v => v >= 0 || "Invalid value"] },
    { key: "maxStockLevel", label: "Max Stock Level", rules: [v => v >= 0 || "Invalid value"] },
];

const dialog = ref(false);
const dialogTitle = ref("");
const isDetailsDialog = ref(false);
const currentEntityId = ref(null);

// NEW: include fkProductId
const editedEntity = ref({
    fkProductId: null,
    quantityInStock: 0,
    minStockLevel: 0,
    maxStockLevel: 0
});

const openAddDialog = () => {
    dialog.value = true;
    dialogTitle.value = `Add New ${entityName}`;
    currentEntityId.value = null;
    isDetailsDialog.value = false;

    editedEntity.value = {
        fkProductId: null,
        quantityInStock: 0,
        minStockLevel: 0,
        maxStockLevel: 0
    };
};

const openEditDialog = (id, entity) => {
    dialog.value = true;
    dialogTitle.value = `Edit ${entityName}`;
    currentEntityId.value = id;
    isDetailsDialog.value = false;

    editedEntity.value = {
        fkProductId: entity.fkProductId,      // NEW
        quantityInStock: entity.quantityInStock ?? 0,
        minStockLevel: entity.minStockLevel ?? 0,
        maxStockLevel: entity.maxStockLevel ?? 0
    };
};

const openDetailsDialog = (id, entity) => {
    openEditDialog(id, entity);
    dialogTitle.value = `Details of ${entityName}`;
    isDetailsDialog.value = true;
};

const deleteEntity = async (id, deleteItem, refresh) => {
    await deleteItem(id);
    await refresh();
};

const close = () => {
    dialog.value = false;
    isDetailsDialog.value = false;
    currentEntityId.value = null;
};

const save = async (createFunction, editFunction, refresh) => {
    if (currentEntityId.value) {
        await editFunction(currentEntityId.value, editedEntity.value);
    } else {
        await createFunction(editedEntity.value);
    }

    dialog.value = false;
    isDetailsDialog.value = false;

    editedEntity.value = {
        fkProductId: null,
        quantityInStock: 0,
        minStockLevel: 0,
        maxStockLevel: 0
    };

    await refresh();
};
</script>

<style scoped>
.table-list {
    width: 100%;
    border-collapse: collapse;
}
.table-list th,
.table-list td {
    border: 1px solid #ddd;
    padding: 8px;
}
.table-list th {
    background-color: #f2f2f2;
    text-align: left;
}
.add-btn {
    display: flex;
    justify-content: flex-end;
    margin: 1rem 0;
}
</style>
