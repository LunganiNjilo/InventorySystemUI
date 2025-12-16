<template>
    <v-app>
        <NavBar />

        <v-main>
            <v-container>
                <v-row>
                    <PageHeader title="Dashboard" text="Welcome to the Inventory Management System" />

                    <DashboardCard title="Categories" :to="{ path: '/categories' }" icon="mdi-folder-outline"
                        color="#fec859" iconColor="#fec859" />
                    <DashboardCard title="Products" :to="{ path: '/products' }" icon="mdi-cart-outline" color="#43b5a0"
                        iconColor="#43b5a0" />
                    <DashboardCard title="Inventory" :to="{ path: '/inventory' }" icon="mdi-package-variant-closed" />
                    <DashboardCard title="Suppliers" :to="{ path: '/suppliers' }" icon="mdi-truck-outline"
                        color="#491d88" iconColor="#491d88" />
                </v-row>
            </v-container>
        </v-main>

        <PageFooter />
    </v-app>
</template>
