<script setup>
import { ref, onMounted } from 'vue';

const brandOptions = ref([]);
const selectedBrand = ref(0);
const selectedList = ref(0);
const listOptions = ref([]);
const operation = ref();
const operation_type = ref(0);
const user_email = ref();

const showSuccessAlert = ref(false);
const showErrorAlert = ref(false);
const ErrorContents = ref([]);

const apiBaseUrl = import.meta.env.VITE_BASE_URL;
const apiSecret = import.meta.env.VITE_API_SECRET;


onMounted(() => {
    populateBrandOptions();
});

const populateBrandOptions = async () => {
    try {
        const formData = new FormData();
        formData.append('api_key', apiSecret);

        // Make a POST request to your API endpoint to get brand options
        const response = await fetch(`${apiBaseUrl}/api/brands/get-brands.php`, {
            method: 'POST',
            body: formData,
        });

        if (response.ok) {
            const data = await response.json();
            brandOptions.value = data; // Assuming your API returns an array of brand options
        } else {
            console.error('Failed to fetch brand options');
        }
    } catch (error) {
        console.error('Error fetching brand options', error);
    }
};

const onBrandChange = () => {
    if (selectedBrand.value != 0) {
        selectedList.value = 0;
        populateListOptions(selectedBrand.value);
    } else {
        // Handle the case where no brand is selected
        listOptions.value = [];
    }
};

const populateListOptions = async (selectedBrandId) => {
    try {
        const formData = new FormData();
        formData.append('api_key', apiSecret);
        formData.append('brand_id', selectedBrandId);

        // Make a second API request to get list options based on the selected brand
        const response = await fetch(`${apiBaseUrl}/api/lists/get-lists.php`, {
            method: 'POST',
            body: formData
            // Add any other configuration for your GET request
        });

        if (response.ok) {
            const data = await response.json();
            listOptions.value = data; // Assuming your API returns an array of list options
        } else {
            console.error('Failed to fetch list options');
        }
    } catch (error) {
        console.error('Error fetching list options', error);
    }
};

const validateFields = () => {
    if (selectedBrand.value == 0) {
        alert('Please select brand to continue');
        return;
    }

    if (selectedList.value == 0) {
        alert('Please select list to continue');
        return;
    }

    if (!operation.value) {
        alert('Please select operation to perform');
        return;
    }

    if (operation_type.value == 0) {
        alert('Please select operation type to perform');
        return;
    }

    if (operation_type.value == 1 && !user_email.value) {
        alert('Missing email info for the operation selected');
        return;
    }


}

const submit = () => {
    //validate the fields
    validateFields();

    if (operation.value == 1) {
        //subscribe operation
        operation_type.value == 1 ? singleSubscribeUser() : multipleSubscribeUser();
        return;
    }

    if (operation.value == 2) {
        //unsubscribe operation
        operation_type.value == 1 ? singleUnSubscribeUser() : multipleUnSubscribeUser();
        return;
    }


};

const singleSubscribeUser = async () => {
    //subscribe single user
    try {
        const formData = new FormData();
        formData.append('api_key', apiSecret);
        formData.append('email', user_email.value);
        formData.append('list', selectedList.value);
        formData.append('boolean', true);

        // Make a POST request to your API endpoint to get brand options
        const response = await fetch(`${apiBaseUrl}/subscribe`, {
            method: 'POST',
            body: formData,
        });

        if (response.ok) {
            const data = await response.text();

            if (data == 1) {
                operation_type.value = 0;
                user_email.value = '';
                showSuccessAlert.value = true;
            } else {
                ErrorContents.value.push({
                    'email': user_email.value,
                    'error': data,
                });
                operation_type.value = 0;
                user_email.value = '';
                showErrorAlert.value = true;
            }

        } else {
            console.error('Failed to submit user info');
        }
    } catch (error) {
        console.error('Error submitting', error);
    }
}

const multipleSubscribeUser = () => {
    //multiplesubscribeUser
}

const singleUnSubscribeUser = () => {
    //unsubscribe single user

}

const multipleUnSubscribeUser = () => {
    //unsubscribe multiple user

}



</script>

<template>
    <div class="  transition-colors duration-300">
        <div class="container mx-auto">
            <div class="bg-white shadow rounded-lg p-6">
                <h1 class="text-xl font-semibold mb-4 text-gray-900 ">Sendly Plug</h1>
                <p class="text-gray-600  mb-6">Plug in for subscribing and unsubscribing for Sendly</p>

                <!-- error alert -->
                <div class="max-w-4xl mx-auto my-4" v-if="showErrorAlert">
                    <div class="bg-red-50 border-l-8 border-red-900">
                        <div class="flex items-center">
                            <div class="p-2">
                                <div class="flex items-center">
                                    <div class="ml-2">
                                        <svg class="h-8 w-8 text-red-900 mr-2 cursor-pointer" @click="showErrorAlert = false"
                                            xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
                                            stroke="currentColor">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                                d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
                                        </svg>
                                    </div>
                                    <p class="px-6 py-4 text-red-900 font-semibold text-base">The following errors occured
                                    </p>
                                </div>
                                <div class="px-16 mb-4">
                                    <li class="text-sm font-bold text-red-500" v-for="(errorObject, index) in ErrorContents"
                                        :key="index">
                                        {{ errorObject.email }} - {{ errorObject.error }}
                                    </li>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <!-- end error alert -->

                <!-- success alert -->
                <div class="max-w-4xl mx-auto my-4" v-if="showSuccessAlert">
                    <div class="bg-green-50 border-l-8 border-green-900">
                        <div class="flex items-center">
                            <div class="p-2">
                                <div class="flex items-center">
                                    <div class="ml-2">
                                        <svg class="h-8 w-8 text-gray-900 mr-2 cursor-pointer"
                                            @click="showSuccessAlert = false" xmlns="http://www.w3.org/2000/svg" fill="none"
                                            viewBox="0 0 24 24" stroke="currentColor">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                                d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
                                        </svg>
                                    </div>
                                    <p class="px-6 py-4 text-gray-900 font-semibold text-base">Operation completed
                                        successfully</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <!-- end success alert -->


                <form class="space-y-4">
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
                        <div>
                            <label for="Brand" class="text-gray-900">Brand</label>
                            <select v-model="selectedBrand" @change="onBrandChange" id="brand"
                                class="border p-2 rounded w-full my-2 text-sm text-gray-500">
                                <option value="0" disabled>Choose Brand ...</option>
                                <option v-for="brand in brandOptions" :key="brand.id" :value="brand.id">{{ brand.name }}
                                </option>

                            </select>
                        </div>
                        <div>
                            <label for="lists" class="text-gray-900">Lists</label>
                            <select id="lists" v-model="selectedList"
                                class="border p-2 rounded w-full my-2 text-sm text-gray-500">
                                <option value="0" disabled>Choose List ...</option>
                                <option v-for="list in listOptions" :key="list.id" :value="list.id">{{ list.name }}</option>
                            </select>
                        </div>
                    </div>

                    <div>
                        <label class="text-gray-900">Choose Operation</label>
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 my-4">

                            <div class="flex items-center mb-4">
                                <input id="subscribe" type="radio" name="countries" value="1" v-model="operation"
                                    class="h-4 w-4 border-gray-300 focus:ring-2 focus:ring-blue-300"
                                    aria-labelledby="subscribe" aria-describedby="subscribe">
                                <label for="subscribe" class="text-sm font-medium text-gray-900 ml-2 block">
                                    Subscribe
                                </label>
                            </div>

                            <div class="flex items-center mb-4">
                                <input id="unsubscribe" type="radio" name="countries" value="0" v-model="operation"
                                    class="h-4 w-4 border-gray-300 focus:ring-2 focus:ring-blue-300"
                                    aria-labelledby="unsubscribe" aria-describedby="unsubscribe">
                                <label for="unsubscribe" class="text-sm font-medium text-gray-900 ml-2 block">
                                    Unsubscribe
                                </label>
                            </div>

                        </div>
                    </div>

                    <div>
                        <label class="text-gray-900">Choose Operation Type</label>
                        <select id="lists" v-model="operation_type"
                            class="border p-2 rounded w-full my-2 text-sm text-gray-500">
                            <option value="0" disabled>Choose type ...</option>
                            <option value="1">Single User</option>
                            <option value="2">Multiple User</option>
                        </select>

                    </div>

                    <div>
                        <div v-show="operation_type == 1">
                            <label for="email" class="text-sm font-medium text-gray-900 block mb-2">Email</label>
                            <input type="email" id="email" v-model="user_email"
                                class="bg-gray-50 border border-gray-300 text-gray-900 sm:text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5"
                                placeholder="Enter email">
                        </div>

                        <div v-show="operation_type == 2">
                            <label for="csv-file" class="text-sm font-medium text-gray-900 block mb-2">Upload CSV</label>
                            <input type="file" id="csv-file"
                                class="bg-gray-50 border border-gray-300 text-gray-900 sm:text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5">
                        </div>

                    </div>
                    <button type="button" @click="submit"
                        class="px-4 py-2 rounded bg-blue-500 text-white hover:bg-blue-600 focus:outline-none transition-colors">
                        Perform Operation
                    </button>
                </form>
            </div>
        </div>

</div></template>