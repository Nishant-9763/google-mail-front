<script setup>
import IntegrationController from "@/controllers/IntegrationController";

//set props thats will come from parent component
const props = defineProps({
    isDialogVisible: {
        type: Boolean,
        required: true,
    },
    gmailIcon: {
        type: String
    }
});

// set emit for value updation in parent function
const emit = defineEmits(["update:isDialogVisible"]);

// variables for json file upload form
const isFormValid = ref(false);
const refForm = ref();
const jsonFile = ref(null);
const selectedFile = ref(null);
const emails = ref([]);
//const emailRegex = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,4}$/;
const isGmailFoldersModelVisible = ref(false);
const loadings = ref([]);

// variables to manage folders view & data
const selectedFolders = ref([]);
const folderList = ref([])
const integrationId = ref(null);

//snackbar to show notifications
const snackbar = ref(false);
const snackbarText = ref("");
const snackbarColor = ref("");

function showSnackbar(type, message, timeout = 3000) {
  snackbarText.value = message;
  snackbar.value = true;
  snackbarColor.value = type;
  setTimeout(() => {
    hideSnackbar();
  }, timeout);
}

function hideSnackbar() {
  snackbar.value = false;
  snackbarText.value = "";
}

//validations
const requiredValidator = (v) => !!v || "This field is required";

const jsonFileValidator = (v) => {
  // Check if a file is selected
  if (!v || v.length == 0 || v == null) {
    return "This field is required";
  }

	// Check if the file size is less than or equal to 2MB and the file type is JSON
	if (v[0].size > 2 * 1024 * 1024 || !v[0].name.toLowerCase().endsWith('.json')) {
		return "File must be less than or equal to 2MB and in JSON format";
	}

  // If everything is okay, return true
  return true;
};

// on call this function, set selected file in a varibale
const handleFileUpload = (event) => {
		const file = event.target.files[0];
		if (file) {
			selectedFile.value = file;
		}
};

// onclick connect button, this function will check form fields validations & if validated then call API
const onSubmit = () => {
  refForm.value?.validate().then(async ({ valid }) => {
    if (valid) {
        load(0);

        const formData = new FormData();
        const emailData = JSON.stringify(emails.value);
        //console.log('emailData---',emailData)
        formData.append("type", 'gmail');
        formData.append("credentials_file", selectedFile.value);
        formData.append("emails", emailData);
        formData.append("integration_type", "EMAIL");

        try {
            const response = await IntegrationController.addGmailIntegration(formData);
            if (response.success == true) {
                const connectResponseData = response.data.connect_response_data;
                if (Object.keys(connectResponseData).length) {
                    Object.entries(connectResponseData).map(([email, rdata]) => {
                        if (rdata.connected) {
                            rdata.email = email;
                            folderList.value.push(rdata)
                        } else {
                            showSnackbar("error", rdata.message);
                        }
                    });
                    
                    //set integration id
                    integrationId.value = response.data.id;

                    // set value true to show folder view in modal
                    isGmailFoldersModelVisible.value = true;
                    dialogVisibleUpdate(false);
                } else {
                    //set integration id
                    integrationId.value = null;
                    //stop loading
                    loadings.value[0] = false;
                    showSnackbar("error", "No data is available for connection");
                }
            } else {
                //set integration id
                integrationId.value = null;
                //stop loading
                loadings.value[0] = false;
                showSnackbar("error", response.message);
            }
        } catch (error) {
            loadings.value[0] = false;
            showSnackbar("error", error);
        }
    }
  });
};

// onclick save button, call API 
const handleSubmit = async () => {
    load(1);
    const selectedFoldersByEmail = {};

    // Loop through selectedFolders to organize them by email address
    // Initialize selectedFoldersByEmail with all email addresses
    folderList.value.forEach((folder, index) => {
        selectedFoldersByEmail[folder.email] = [];
    });

    // Loop through selectedFolders to organize them by email address
    selectedFolders.value.forEach(selectedFolder => {
        const [emailIndex, label] = selectedFolder.split("-");
        const email = folderList.value[emailIndex].email;

        // Push the label to the array corresponding to the email address
        selectedFoldersByEmail[email].push(label);
    });

    //console.log('test---', selectedFoldersByEmail)
    const payload = {
        "email_labels": selectedFoldersByEmail
    }
    try {
        const response = await IntegrationController.addGmailIntegrationFolders(payload, integrationId.value);
        //console.log('response---',response)
        if (response.success === true) {
            dialogGmailFoldersVisibleUpdate(false);
            showSnackbar("success", "Gmail account integrated successfully");
        } else {
            //stop loading
            loadings.value[1] = false;
            showSnackbar("error", response.message);
        }
    } catch (error) {
        //stop loading
        loadings.value[1] = false;
        showSnackbar("error", error);
    }
}

//handle loading
const load = (i) => {
  loadings.value[i] = true;
};

// Function for modal close & data reset
const dialogVisibleUpdate = (val) => {
    emit("update:isDialogVisible", val);
    //Reset form data
    jsonFile.value = null;
    selectedFile.value = null;
    emails.value = [];
    //stop loading
    loadings.value[0] = false;
};

const dialogGmailFoldersVisibleUpdate = (val) => {
    isGmailFoldersModelVisible.value = val;
    integrationId.value = null;
    folderList.value = [];
    selectedFolders.value = [];
    loadings.value[1] = false;
};

//manage status colour
const resolveStatus = status => {
  if (status === true)
    return {
      text: 'Connected',
      color: 'success',
    }
  if (status === false)
    return {
      text: 'Connection Failed',
      color: '#F57F3C',
    }
}
</script>

<template>
<VDialog :model-value="props.isDialogVisible" max-width="550" class="gmail-integration" @update:model-value="dialogVisibleUpdate" persistent>
    <VCard>
        <DialogCloseBtn variant="text" size="default" class="gmail-dialog-close" @click="dialogVisibleUpdate(false)" />
        <VCardText class="pt-5">
            <VForm ref="refForm" v-model="isFormValid" @submit.prevent="onSubmit">
                <span class="font-20-dark">Gmail Authentication</span>

                <div class="mt-2">
                    <VImg :src="gmailIcon" alt="Gmail" :width="70" :height="70" class="mb-2" />
                </div>
                <VCardItem class="px-0 pt-1">
                    <VCardTitle>Upload credentials.json file for successful connection</VCardTitle>
                </VCardItem>
                <VRow>
                    <VCol cols="12" class="pb-0">
                        <VFileInput v-model="jsonFile" label="File" accept=".json" class="gmail-file mb-3" density="compact" :rules="[requiredValidator, jsonFileValidator]" @change="handleFileUpload" />
                    </VCol>
                    <VCol cols="12" class="pb-0">
                        <VCombobox
                            v-model="emails"
                            prepend-inner-icon="ri-mail-line"
                            label="Email"
                            chips
                            clearable
                            multiple
                            closable-chips
                            clear-icon="ri-close-circle-line"
                            placeholder="Enter Email ID"
                            density="compact"
                            />
                    </VCol>
                    <VCol cols="12">
                        <span class="mt-2">Lorem ipsum dolor sit amet consectetur. Convallis a quis neque nec mi metus et elementum feugiat.</span>
                    </VCol>
                    <VCol class="text-end">
                        <VBtn :loading="loadings[0]" :disabled="loadings[0]" color="primary" size="small" type="submit">
                            <VIcon start icon="ri-checkbox-circle-line" />
                            Connect
                        </VBtn>
                    </VCol>
                </VRow>
            </VForm>
        </VCardText>
    </VCard>
</VDialog>
<VDialog :model-value="isGmailFoldersModelVisible" class="v-dialog-xl gmail-integration" @update:model-value="dialogGmailFoldersVisibleUpdate" persistent>
    <VCard>
        <DialogCloseBtn variant="text" size="default" class="gmail-dialog-close" @click="dialogGmailFoldersVisibleUpdate(false)" />
        <VCardText class="pt-5">
            <span class="font-20-dark">Gmail Folder Selection</span>

            <div class="mt-2">
                <VImg :src="gmailIcon" alt="Gmail" :width="70" :height="70" class="mb-2" />
            </div>
            <VCardItem class="px-0 pt-1">
                <VCardTitle>Please select Gmail folders for data connection</VCardTitle>
            </VCardItem>
            <VRow>
                <div class="px-3 folder-list pt-2" v-for="(folder, findex) in folderList" :key="findex">
                    <div class="d-flex align-items-center">
                        <div class="text-body-1 font-weight-medium mb-3 text-high-emphasis me-2">
                            {{ `${findex+1}. ${folder.email}` }}
                        </div>
                        <VChip
                            v-bind="resolveStatus(folder.connected)"
                            size="small"
                        />
                    </div>
                    
                    <div v-if="folder.connected" class="demo-space-x text-body-1">
                        <VCheckbox v-for="(flabel, index) in folder.labels" :key="`${findex}-${index}`"  v-model="selectedFolders" :label="flabel" :value="`${findex}-${flabel}`" />
                    </div>
                    
                    <p v-if="!folder.connected">{{ folder.message }}</p>
                </div>

                <VCol class="text-end">
                    <VBtn :loading="loadings[1]" :disabled="loadings[1]" color="primary" size="small" type="button" @click="handleSubmit">
                        <VIcon start icon="ri-checkbox-circle-line" />
                        Save
                    </VBtn>
                </VCol>
            </VRow>
        </VCardText>
    </VCard>
</VDialog>
<template>
    <VSnackbar
      v-model="snackbar"
      :color="snackbarColor"
      location="top end"
      variant="flat"
      transition="scroll-y-reverse-transition"
    >
      {{ snackbarText }}
      <template #actions>
        <VBtn color="white" variant="text" @click="snackbar = false">
          Close
        </VBtn>
      </template>
    </VSnackbar>
</template>
</template>

<style lang="scss">
.gmail-dialog-close {
    color: red !important;
}
.gmail-file{
    position: relative;
    .v-input__prepend{
        position: absolute;
        left: 15px;
        top: 12px;
    }
    .v-field--appended{
        padding-left: 28px;
    }
}
.folder-list{
    border-top: 1px solid #d5d5d5;
    .v-label{
        font-size: 12px;
    }
    .v-input{
        margin-block-start: 8px;
    }
    .v-label{
        padding-top: 4px;
    }
}
.gmail-integration{
    .v-btn__content{
        align-items: unset;
    }
}
</style>
