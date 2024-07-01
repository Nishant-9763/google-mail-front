<script setup>
// import IntegrationController from "@/controllers/IntegrationController";
// import GmailAuthentication from "@/views/integration/GmailAuthentication.vue";
// import OutlookAuthentication from "@/views/integration/OutlookAuthentication.vue";
// import connectedDisabledIcon from "@images/integration/connect-disabled.png";
// import connectedIcon from "@images/integration/connect.png";
import gmailIcon from "@images/integrations/gmail.png";
// import notConnectedDisabledIcon from "@images/integration/not-connect-disabled.png";
// import notConnectedIcon from "@images/integration/not-connect.png";
import outlookIcon from "@images/integrations/outlook.png";

const route = useRoute();
const router = useRouter();

const breadcrumbText = ref('');
const accountType = ref('');
const isGmailAuthenticationModelVisible = ref(false);
const isOutlookAuthenticationModelVisible = ref(false);

//static integration card data
const integrationData = ref([]);

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

onMounted(() => {
  breadcrumbText.value = convertSlugToText(route.params.slug);

  fetchIntegrationEmailsData();
})

// convert into slug into normal text format
const convertSlugToText = (slug) => {
  // Replace dashes with spaces
  let text = slug.replace(/-/g, ' ');

  // Capitalize the first letter of each word
  text = text.replace(/\b\w/g, (char) => char.toUpperCase());

  return text;
}
const dummyData = [
  {
    id:1,
    image: gmailIcon,
    provider: "Google",
    description: "rtygcvghjk"
  },
  {
    id:2,
    image: outlookIcon,
    provider: "Outlook",
    description: "rtyui"
  }
]

//fetch integration emails list
const fetchIntegrationEmailsData = async () => {
  try {
    let response
    // const response //= await IntegrationController.fetchIntegrationEmailsData();
    if (response.success == true) {
      integrationData.value = response.data;
      const data = integrationData.value.map((ele, index) => {
        ele.accountType = ele.provider;

        if (ele.accountType == 'gmail') {
          ele.image = gmailIcon;
        }

        if (ele.accountType == 'outlook') {
          ele.image = outlookIcon;
        }

        ele.isConnected = ele.status_json != null && Object.values(ele.status_json).some(status => status === 'CONNECTED');

        ele.connectedEmails = ele.status_json ? Object.keys(ele.status_json).filter(email => ele.status_json[email] === "CONNECTED") : [];

        ele.description = "Lorem ipsum dolor sit amet consectetur. Convallis a quis neque nec mi metus et elementum feugiat.";

        return ele;
      });
    } else {
      showSnackbar("error", response.message);
    }
  } catch (error) {
    showSnackbar("error", error);
  }
}

//open authentication screen
const gotoAuthentication = (aType, isConnected) => {
  if (!isConnected) {
    accountType.value = aType;

    if (aType == 'gmail') {
      isGmailAuthenticationModelVisible.value = true;
      isOutlookAuthenticationModelVisible.value = false;
    }

    if (aType == 'outlook') {
      isGmailAuthenticationModelVisible.value = false;
      isOutlookAuthenticationModelVisible.value = true;
    }
  }
}

//change connect to connect
const handleConnet = async (integration) => {
  try {
    if (integration.accountType == 'gmail') {
      let response;
      // const response = await IntegrationController.connectIntegrationEmail(integration.id);
      if (response.success == true) {
        showSnackbar("success", 'Integration email conneted successfully.');
        integration.isConnected = true;
      } else {
        showSnackbar("error", response.message);
      }
    }
  } catch (error) {
    showSnackbar("error", error);
  }
}

//change connect to disconnect
const handleDisconnet = async (integration) => {
  try {
    if (integration.accountType == 'gmail') {
      let response;
      // const response = await IntegrationController.disconnectIntegrationEmail(integration.id);
      if (response.success == true) {
        showSnackbar("success", 'Integration email disconneted successfully.');
        integration.isConnected = false;
      } else {
        showSnackbar("error", response.message);
      }
    }
  } catch (error) {
    showSnackbar("error", error);
  }
}

//go to integration list page
const gotoIntegration = () => {
  router.push("/integration");
}

</script>

<template>
  <h4 class="text-h4 mb-4">Integration / {{ breadcrumbText }}</h4>
  <VRow>
    <VCol class="text-start">
      <VBtn color="primary" size="small" type="button" @click="gotoIntegration">
        Back
      </VBtn>
    </VCol>
  </VRow>
  <VRow class="match-height">
    <VCol cols="12" md="4" v-for="integration in dummyData" :key="integration.id" class="px-5 mb-4">
      <VCard class="integration-single-card" @click="gotoAuthentication(integration.provider, integration.isConnected)">
        <VCardText class="d-flex gap-y-2 flex-column">
          <div class="d-flex">
            <VImg :src="integration.image" :alt="integration.provider" :width="90" :height="90" class="mb-2 me-5" />
            <div class="text-center d-flex">
              <VChip :color="integration.isConnected ? 'success' : '#F57F3C'" variant="elevated" class="rounded me-2">
                {{ integration.isConnected ? 'Connected' : 'Not Connected' }}
              </VChip>

              <VImg :src="integration.isConnected ? connectedIcon : connectedDisabledIcon" :alt="integration.provider"
                :width="integration.isConnected ? 50 : 30" :height="integration.isConnected ? 50 : 30"
                :class="integration.isConnected ? 'connected-icon me-1' : 'not-connected-icon me-1'"
                @click.stop="handleDisconnet(integration)" />

              <VImg :src="integration.isConnected ? notConnectedDisabledIcon : notConnectedIcon"
                :alt="integration.provider" :width="integration.isConnected ? 30 : 50"
                :height="integration.isConnected ? 30 : 50"
                :class="integration.isConnected ? 'not-connected-icon' : 'connected-icon'"
                @click.stop="handleConnet(integration)" />
            </div>
          </div>

          <h2 class="font-weight-medium px-1 mb-1 text-capitalize">
            {{ integration.provider }} Account
          </h2>
          <h6 class="text-xs font-weight-medium px-1">{{ integration?.connectedEmails?.length > 0 ?
    integration.connectedEmails.join(", ") : '---' }}</h6>
          <div class="text-base px-1">
            <p class="mb-0 text-base">
              {{ integration.description }}
            </p>
          </div>
        </VCardText>
      </VCard>
    </VCol>
  </VRow>
  <!-- <GmailAuthentication v-if="accountType == 'gmail'" v-model:is-dialog-visible="isGmailAuthenticationModelVisible" :gmailIcon="gmailIcon" />
  <OutlookAuthentication v-if="accountType == 'outlook'" v-model:is-dialog-visible="isOutlookAuthenticationModelVisible" :outlookIcon="outlookIcon" /> -->

  <template>
    <VSnackbar v-model="snackbar" :color="snackbarColor" location="top end" variant="flat"
      transition="scroll-y-reverse-transition">
      {{ snackbarText }}
      <template #actions>
        <VBtn color="white" variant="text" @click="snackbar = false">
          Close
        </VBtn>
      </template>
    </VSnackbar>
  </template>
</template>
<style lang="scss" scoped>
.integration-single-card {
  .connected-icon {
    margin-top: -7px;
    cursor: pointer;
  }

  .not-connected-icon {
    margin-top: 0px;
    cursor: pointer;
  }
}
</style>
