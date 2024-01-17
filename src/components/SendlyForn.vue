<script setup>
import { ref, onMounted } from 'vue';

const brandOptions = ref([]);
const selectedBrand = ref(0);
const selectedList = ref(0);
const listOptions = ref([]);
const operation = ref();
const operation_type = ref(0);
const user_email = ref();
const csv_data = ref();
const isProcessing = ref(false);

const showSuccessAlert = ref(false);
const showErrorAlert = ref(false);
const ErrorContents = ref([]);

const apiBaseUrl = import.meta.env.VITE_BASE_URL;
const apiSecret = import.meta.env.VITE_API_SECRET;
const envBrandId = import.meta.env.VITE_BRAND_ID;
const envBrandName = import.meta.env.VITE_BRAND_NAME;



onMounted(() => {
    //if brand name and id declared then load them only
    if (envBrandId && envBrandName) {
        brandOptions.value = [
            { id: envBrandId, name: envBrandName }
        ];
        selectedBrand.value = envBrandId;
        //load the list for that brand
        populateListOptions(selectedBrand.value);
    } else {
        populateBrandOptions();
    }
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
        return false;
    }

    if (selectedList.value == 0) {
        alert('Please select list to continue');
        return false;
    }

    if (!operation.value) {
        alert('Please select operation to perform');
        return false;
    }

    if (operation_type.value == 0) {
        alert('Please select operation type to perform');
        return false;
    }

    if (operation_type.value == 1 && !user_email.value) {
        alert('Missing email info for the operation selected');
        return false;
    }

    if (operation_type.value == 2 && !csv_data.value) {
        alert('Missing data from the csv file');
        return false;
    }


}

const submit = async () => {

    if (validateFields()) {
        //clears all alerts
        showErrorAlert.value = false;
        ErrorContents.value = [];
        showSuccessAlert.value = false;
        isProcessing.value = true;

        
        if (operation.value == 1) {
            //subscribe operation
            if (operation_type.value == 1) {
                const response = await singleSubscribeUser(user_email.value, selectedList.value);

                if (response == 1) {
                    operation_type.value = 0;
                    user_email.value = '';
                    showSuccessAlert.value = true;
                    isProcessing.value = false;

                } else {
                    ErrorContents.value.push({
                        'email': user_email.value,
                        'error': response,
                    });
                    operation_type.value = 0;
                    user_email.value = '';
                    showErrorAlert.value = true;
                    isProcessing.value = false;

                }

            } else {
                await multipleSubscribeUser();
                if (ErrorContents.value.length > 0) {
                    showErrorAlert.value = true;
                    operation_type.value = 0;
                    csv_data.value = '';
                } else {
                    showSuccessAlert.value = true;
                    operation_type.value = 0;
                    csv_data.value = '';
                }
                isProcessing.value = false;

            }
            return;
        }

        if (operation.value == 2) {
            //unsubscribe operation
            if (operation_type.value == 1) {
                const response = await singleUnSubscribeUser(user_email.value, selectedList.value);
                if (response == 1) {
                    operation_type.value = 0;
                    user_email.value = '';
                    showSuccessAlert.value = true;
                    isProcessing.value = false;

                } else {
                    ErrorContents.value.push({
                        'email': user_email.value,
                        'error': response,
                    });
                    operation_type.value = 0;
                    user_email.value = '';
                    showErrorAlert.value = true;
                    isProcessing.value = false;
                }

            } else {
                await multipleUnSubscribeUser();
                if (ErrorContents.value.length > 0) {
                    showErrorAlert.value = true;
                    operation_type.value = 0;
                    csv_data.value = '';
                } else {
                    showSuccessAlert.value = true;
                    operation_type.value = 0;
                    csv_data.value = '';
                }
                isProcessing.value = false;

            }
            return;

        }
    }

};

const singleSubscribeUser = async (email, list_id, name = null) => {
    //subscribe single user
    try {
        const formData = new FormData();
        formData.append('api_key', apiSecret);
        formData.append('email', email);
        formData.append('list', list_id);
        formData.append('name', name);
        formData.append('boolean', true);

        // Make a POST request to your API endpoint to get brand options
        const response = await fetch(`${apiBaseUrl}/subscribe`, {
            method: 'POST',
            body: formData,
        });

        if (response.ok) {
            const data = await response.text();

            return data;
        } else {
            console.error('Failed to submit user info');
        }
    } catch (error) {
        console.error('Error submitting', error);
    }
}

const multipleSubscribeUser = async () => {
    //multiplesubscribeUser
    for (const data of csv_data.value) {
        var email, name;
        email = data.email;
        if (data.name) { //because sometimes can be unvailable(not required)
            name = data.name;
        }

        const response = await singleUnSubscribeUser(email, selectedList.value, name);
        if (response != 1) {
            ErrorContents.value.push({
                'email': user_email.value,
                'error': response,
            });
        }
    }
}

const singleUnSubscribeUser = async (email, list_id, name = null) => {
    //unsubscribe single user
    try {
        const formData = new FormData();
        formData.append('api_key', apiSecret);
        formData.append('email', email);
        formData.append('list', list_id);
        formData.append('name', name);
        formData.append('boolean', true);

        // Make a POST request to your API endpoint to get brand options
        const response = await fetch(`${apiBaseUrl}/unsubscribe`, {
            method: 'POST',
            body: formData,
        });

        if (response.ok) {
            const data = await response.text();

            return data;

        } else {
            console.error('Failed to submit user info');
        }
    } catch (error) {
        console.error('Error submitting', error);
    }

}

const multipleUnSubscribeUser = async () => {
    //unsubscribe multiple user

    for (const data of csv_data.value) {
        var email, name;
        email = data.email;
        if (data.name) { //because sometimes can be unvailable(not required)
            name = data.name;
        }

        const response = await singleUnSubscribeUser(email, selectedList.value, name);
        if (response != 1) {
            ErrorContents.value.push({
                'email': user_email.value,
                'error': response,
            });
        }
    }

}



</script>

<template>
    <div class="transition-colors duration-300 ">
        <div class="container mx-auto">
            <div class="p-6 bg-white rounded-lg shadow">
                <h1 class="mb-4 text-xl font-semibold text-gray-900 ">Sendy Plug</h1>
                <p class="mb-6 text-gray-600">Plug in for subscribing and unsubscribing for Sendy</p>

                <!-- error alert -->
                <div class="max-w-4xl mx-auto my-4" v-if="showErrorAlert">
                    <div class="border-l-8 border-red-900 bg-red-50">
                        <div class="flex items-center">
                            <div class="p-2">
                                <div class="flex items-center">
                                    <div class="ml-2">
                                        <svg class="w-8 h-8 mr-2 text-red-900 cursor-pointer"
                                            @click="showErrorAlert = false" xmlns="http://www.w3.org/2000/svg" fill="none"
                                            viewBox="0 0 24 24" stroke="currentColor">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                                d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
                                        </svg>
                                    </div>
                                    <p class="px-6 py-4 text-base font-semibold text-red-900">Operation completed with the
                                        following errors
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
                    <div class="border-l-8 border-green-900 bg-green-50">
                        <div class="flex items-center">
                            <div class="p-2">
                                <div class="flex items-center">
                                    <div class="ml-2">
                                        <svg class="w-8 h-8 mr-2 text-gray-900 cursor-pointer"
                                            @click="showSuccessAlert = false" xmlns="http://www.w3.org/2000/svg" fill="none"
                                            viewBox="0 0 24 24" stroke="currentColor">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                                d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z" />
                                        </svg>
                                    </div>
                                    <p class="px-6 py-4 text-base font-semibold text-gray-900">Operation completed
                                        successfully</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <!-- end success alert -->


                <form class="space-y-4"  @submit.prevent="submit">
                    <div class="grid grid-cols-1 gap-4 mb-4 md:grid-cols-2">
                        <div>
                            <label for="Brand" class="text-gray-900">Brand</label>
                            <select v-model="selectedBrand" @change="onBrandChange" id="brand"
                                class="w-full p-2 my-2 text-sm text-gray-500 border rounded">
                                <option value="0" disabled>Choose Brand ...</option>
                                <option v-for="brand in brandOptions" :key="brand.id" :value="brand.id">{{ brand.name }}
                                </option>

                            </select>
                        </div>
                        <div>
                            <label for="lists" class="text-gray-900">Lists</label>
                            <select id="lists" v-model="selectedList"
                                class="w-full p-2 my-2 text-sm text-gray-500 border rounded">
                                <option value="0" disabled>Choose List ...</option>
                                <option v-for="list in listOptions" :key="list.id" :value="list.id">{{ list.name }}</option>
                            </select>
                        </div>
                    </div>

                    <div>
                        <label class="text-gray-900">Choose Operation</label>
                        <div class="grid grid-cols-1 gap-4 my-4 md:grid-cols-2">

                            <div class="flex items-center mb-4">
                                <input id="subscribe" type="radio" name="countries" value="1" v-model="operation"
                                    class="w-4 h-4 border-gray-300 focus:ring-2 focus:ring-blue-300"
                                    aria-labelledby="subscribe" aria-describedby="subscribe">
                                <label for="subscribe" class="block ml-2 text-sm font-medium text-gray-900">
                                    Subscribe
                                </label>
                            </div>

                            <div class="flex items-center mb-4">
                                <input id="unsubscribe" type="radio" name="countries" value="2" v-model="operation"
                                    class="w-4 h-4 border-gray-300 focus:ring-2 focus:ring-blue-300"
                                    aria-labelledby="unsubscribe" aria-describedby="unsubscribe">
                                <label for="unsubscribe" class="block ml-2 text-sm font-medium text-gray-900">
                                    Unsubscribe
                                </label>
                            </div>

                        </div>
                    </div>

                    <div>
                        <label class="text-gray-900">Choose Operation Type</label>
                        <select id="lists" v-model="operation_type"
                            class="w-full p-2 my-2 text-sm text-gray-500 border rounded">
                            <option value="0" disabled>Choose type ...</option>
                            <option value="1">Single User</option>
                            <option value="2">Multiple User</option>
                        </select>

                    </div>

                    <div>
                        <div v-show="operation_type == 1">
                            <label for="email" class="block mb-2 text-sm font-medium text-gray-900">Email</label>
                            <input type="email" id="email" v-model="user_email" @keyup.enter="submit"
                                class="bg-gray-50 border border-gray-300 text-gray-900 sm:text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5"
                                placeholder="Enter email">
                        </div>

                        <div v-show="operation_type == 2">
                            <label for="csv-file" class="block mb-2 text-sm font-medium text-gray-900">Upload CSV</label>
                            <!-- <input type="file" id="csv-file"
                                class="bg-gray-50 border border-gray-300 text-gray-900 sm:text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5"> -->
                            <vue-csv-import v-model="csv_data"
                                :fields="{ name: { required: false, label: 'Name' }, email: { required: true, label: 'Email' } }">
                                <vue-csv-toggle-headers class="my-1 text-sm"></vue-csv-toggle-headers>
                                <vue-csv-errors></vue-csv-errors>
                                <vue-csv-input
                                    class="bg-gray-50 border border-gray-300 text-gray-900 sm:text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5">

                                </vue-csv-input>
                                <vue-csv-map class="p-4 mt-4 border border-gray-300 rounded-md "></vue-csv-map>
                            </vue-csv-import>
                        </div>

                    </div>
                    <button type="button" @click="submit" v-show="!isProcessing"
                        class="w-full px-4 py-2 text-white transition-colors bg-blue-500 rounded hover:bg-blue-600 focus:outline-none">
                        PERFORM OPERATION
                    </button>

                    <button type="button" v-show="isProcessing" class="w-full text-white bg-gray-900 hover:bg-gray-800 focus:ring-4 focus:outline-none focus:ring-gray-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-gray-600 dark:hover:bg-gray-700 dark:focus:ring-gray-800
                    animate-pulse">PROCESSING ...
                    </button>
                </form>
            </div>
        </div>

    </div>
</template>

<style scoped>
select {
    border-width: 1px;
    padding: 0.5rem;
    border-radius: 0.25rem;
    margin-top: 0.5rem;
    margin-bottom: 0.5rem;
    font-size: 0.875rem;
    line-height: 1.25rem;
}
</style>