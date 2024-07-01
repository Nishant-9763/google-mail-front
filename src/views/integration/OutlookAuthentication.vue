<script setup>
//set props thats will come from parent component
const props = defineProps({
    isDialogVisible: {
        type: Boolean,
        required: true,
    },
    outlookIcon: {
        type: String
    }
});

// set emit for value updation in parent function
const emit = defineEmits(["update:isDialogVisible"]);

// dummy folder list
const folderList = [{
        id: 1,
        name: 'Inbox'
    },
    {
        id: 2,
        name: 'Starred'
    },
    {
        id: 3,
        name: 'Important'
    },
    {
        id: 4,
        name: 'Social'
    },
    {
        id: 5,
        name: 'Updates'
    },
    {
        id: 6,
        name: 'Forums'
    },
    {
        id: 7,
        name: 'Promotions'
    },
    {
        id: 8,
        name: 'Spam'
    }
]

//variable for outlook account
const isFormValid = ref(false);
const refForm = ref();
const clientId = ref(null);
const clientSecret = ref(null);
const emails = ref([]);
const emailRegex = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,4}$/;

// variables to manage folders view & data
const isShowFolder = ref(false);
const selectedFolders = ref([]);

//validations
const requiredValidator = (v) => !!v || "This field is required";

// onclick connect button, this function will check form fields validations & if validated then call API
const onSubmit = () => {
  refForm.value?.validate().then(({ valid }) => {
    if (valid) {
        const emailData = JSON.stringify(emails.value);

        const payload = {
            clientId: clientId.value,
            clientSecret: clientSecret.value,
            email: emailData
        }
        console.log('payload------',payload)
        // set value true to show folder view in modal
        isShowFolder.value = true;
    }
  });
};

// onclick save button, call API 
const handleSubmit = () => {
    console.log('test---', selectedFolders.value)
    dialogVisibleUpdate(false)
}

// Function for modal close & data reset
const dialogVisibleUpdate = (val) => {
    emit("update:isDialogVisible", val);
    setTimeout(() => {
        selectedFolders.value = [];
        isShowFolder.value = false;
    }, 1000);
};
</script>
<template>
<VDialog :model-value="props.isDialogVisible" max-width="550" class="outlook-integration" @update:model-value="dialogVisibleUpdate" persistent>
    <VCard>
        <DialogCloseBtn variant="text" size="default" class="outlook-dialog-close" @click="dialogVisibleUpdate(false)" />
        <VCardText v-if="!isShowFolder" class="pt-5">
            <VForm ref="refForm" v-model="isFormValid" @submit.prevent="onSubmit">
                <span class="font-20-dark">Outlook Authentication</span>

                <div class="mt-2">
                    <VImg :src="outlookIcon" alt="Outlook" :width="70" :height="70" class="mb-2" />
                </div>
                <VCardItem class="px-0 pt-1 pb-2">
                    <VCardTitle>Input following credentials for successful connection</VCardTitle>
                </VCardItem>
                <VRow>
                    <VCol cols="12" class="pb-0">
                        <label class="text-h6 primary-black">Client Id</label>

                        <VTextField v-model="clientId" type="text" placeholder="Enter Client ID Here" density="compact" :rules="[requiredValidator]" />
                    </VCol>
                    <VCol cols="12" class="pb-0">
                        <label class="text-h6 primary-black">Client Secret</label>

                        <VTextField v-model="clientSecret" type="text" placeholder="Enter Client Secret Here" density="compact" :rules="[requiredValidator]" />
                    </VCol>
                    <VCol cols="12" class="pb-0">
                        <label class="text-h6 primary-black">Email ID</label>
                        <VCombobox
                            v-model="emails"
                            prepend-inner-icon="ri-mail-line"
                            chips
                            clearable
                            multiple
                            closable-chips
                            clear-icon="ri-close-circle-line"
                            placeholder="Enter Email ID Here"
                            density="compact"
                        />
                    </VCol>
                    <VCol cols="12">
                        <span class="mt-2">Lorem ipsum dolor sit amet consectetur. Convallis a quis neque nec mi metus et elementum feugiat.</span>
                    </VCol>
                    <VCol class="text-end">
                        <VBtn color="primary" size="small" type="submit" >
                            <VIcon start icon="ri-checkbox-circle-line" />
                            Connect
                        </VBtn>
                    </VCol>
                </VRow>
            </VForm>
        </VCardText>
        <VCardText v-if="isShowFolder" class="pt-5">
            <span class="font-20-dark">Outlook Folder Selection</span>

            <div class="mt-2">
                <VImg :src="outlookIcon" alt="outlook" :width="70" :height="70" class="mb-2" />
            </div>
            <VCardItem class="px-0 pt-1">
                <VCardTitle>Please select Outlook folders for data connection</VCardTitle>
            </VCardItem>
            <VRow>
                <div class="demo-space-x px-3">
                    <VCheckbox v-for="folder in folderList" :key="folder.id" v-model="selectedFolders" :label="folder.name" :value="folder" />
                </div>

                <VCol class="text-end">
                    <VBtn color="primary" size="small" type="button" @click="handleSubmit">
                        <VIcon start icon="ri-checkbox-circle-line" />
                        Save
                    </VBtn>
                </VCol>
            </VRow>
        </VCardText>
    </VCard>
</VDialog>
</template>

<style lang="scss" scoped>
.outlook-dialog-close {
    color: red !important;
}
.outlook-integration{
    .v-btn__content{
        align-items: unset;
    }
}
</style>
