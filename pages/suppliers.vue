<template>
    <NavBar />
    <v-container>
        <v-row>
            <PageHeader :title="pageTitle" :text="pageDescription" />
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
                :deleteEntity="(id) => deleteEntity(id, deleteItem, refresh, data)"
            />

            <v-dialog v-model="dialog" persistent max-width="600px">
                <v-card>
                    <v-card-title>
                        <span class="headline">{{ dialogTitle }}</span>
                    </v-card-title>

                    <v-card-text>
                        <v-container>
                            <v-row>
                                <v-col cols="12" v-for="(field, index) in entityFields" :key="index">
                                    <v-text-field
                                        v-model="editedEntity[field.key]"
                                        :label="field.label"
                                        :rules="field.rules"
                                        :placeholder="field.placeholder"
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
                            @click="() => save(create, edit, refresh)"
                            v-if="!isDetailsDialog"
                        >
                            Save
                        </v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>
        </template>
    </DataFetcher>

    <PageFooter />
</template>

<script setup>
import { ref } from "vue";

const pageTitle = 'Suppliers';
const pageDescription = 'Manage your suppliers here.';
const entityName = 'Supplier';

const headers = [
    'Supplier Name',
    'Address',
    'Contact Email',
    'Contact First Name'
];

const entityKeys = [
    'supplierName',
    'supplierAddress',
    'contactEmail',
    'contactFirstName'
];

const apiBaseUri = 'http://localhost:8080/api/Suppliers';

const entityFields = [
    { key: 'supplierName', label: 'Supplier Name' },
    { key: 'supplierAddress', label: 'Address' },
    { key: 'contactEmail', label: 'Email' },
    { key: 'contactFirstName', label: 'First Name' },
    { key: 'contactLastName', label: 'Last Name' }
];

const dialog = ref(false);
const dialogTitle = ref('');
const isDetailsDialog = ref(false);
const currentEntityId = ref(null);

const editedEntity = ref({
    supplierName: '',
    supplierAddress: '',
    contactEmail: '',
    contactFirstName: '',
    contactLastName: ''
});

const openAddDialog = () => {
    dialog.value = true;
    dialogTitle.value = `Add New ${entityName}`;

    editedEntity.value = {
        supplierName: '',
        supplierAddress: '',
        contactEmail: '',
        contactFirstName: '',
        contactLastName: ''
    };
};

const openEditDialog = (id, entity) => {
    dialogTitle.value = `Edit ${entityName}`;
    editedEntity.value = { ...entity };
    currentEntityId.value = id;
    dialog.value = true;
};

const openDetailsDialog = (id, entity) => {
    dialogTitle.value = `Details of ${entityName}`;
    editedEntity.value = { ...entity };
    currentEntityId.value = id;
    dialog.value = true;
    isDetailsDialog.value = true;
};

const deleteEntity = async (id, deleteItem, refresh, data) => {
    await deleteItem(id);
    await refresh();

    // ⭐ THE ONLY FIX YOU NEEDED:
    data.result = [...data.result];
};

const close = () => {
    dialog.value = false;
    isDetailsDialog.value = false;
};

const save = async (createFunction, editFunction, refresh) => {
    if (currentEntityId.value) {
        await editFunction(currentEntityId.value, editedEntity.value);
    } else {
        await createFunction(editedEntity.value);
    }

    dialog.value = false;
    close();
    refresh();
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
