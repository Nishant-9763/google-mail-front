<script setup>
// import atsIcon from "@images/integration/ats.png";
// import calendarIcon from "@images/integration/calendar.png";
// import customApiIcon from "@images/integration/custom-api.png";
// import emailIcon from "@images/integration/email.png";
// import jobBoardIcon from "@images/integration/job-board.png";
// import starConnectionIcon from "@images/integration/star-connection.png";

// import atsWhiteIcon from "@images/integration/ats-white.png";
// import calendarWhiteIcon from "@images/integration/calendar-white.png";
// import customApiWhiteIcon from "@images/integration/custom-api-white.png";
// import emailWhiteIcon from "@images/integration/email-white.png";
// import jobBoardWhiteIcon from "@images/integration/job-board-white.png";
// import starConnectionWhiteIcon from "@images/integration/star-connection-white.png";

const router = useRouter();

//static integration card data
const integrationData = ref([
    {
        id: 1,
        title: "Email",
        description: "Lorem ipsum dolor sit amet consectetur. Convallis a quis neque nec mi metus et elementum feugiat.",
        // image: emailIcon,
        // hoverImage: emailWhiteIcon,
        isHovered: false
    },
    // {
    //     id: 2,
    //     title: "ATS",
    //     description: "Lorem ipsum dolor sit amet consectetur. Convallis a quis neque nec mi metus et elementum feugiat.",
    //     image: atsIcon,
    //     hoverImage: atsWhiteIcon,
    //     isHovered: false
    // },
    // {
    //     id: 3,
    //     title: "Job Board",
    //     description: "Lorem ipsum dolor sit amet consectetur. Convallis a quis neque nec mi metus et elementum feugiat.",
    //     image: jobBoardIcon,
    //     hoverImage: jobBoardWhiteIcon,
    //     isHovered: false
    // },
    // {
    //     id: 4,
    //     title: "Calendar",
    //     description: "Lorem ipsum dolor sit amet consectetur. Convallis a quis neque nec mi metus et elementum feugiat.",
    //     image: calendarIcon,
    //     hoverImage: calendarWhiteIcon,
    //     isHovered: false
    // },
    // {
    //     id: 1,
    //     title: "Custom API",
    //     description: "Lorem ipsum dolor sit amet consectetur. Convallis a quis neque nec mi metus et elementum feugiat.",
    //     image: customApiIcon,
    //     hoverImage: customApiWhiteIcon,
    //     isHovered: false
    // }
]);

//manage mouse over effect on card
const handleMouseOver = (integration) => {
    integration.isHovered = true;
}

//manage mounse leave effect on card to set in bydefault preview
const handleMouseLeave = (integration) => {
    integration.isHovered = false;
}

//onclick card go to single page of integration
const gotoSinglePage = (urlText) => {
  if (urlText == 'Email') {
    const slugUrl = convertUrlToSlug(urlText);

    router.push("/integration/view/"+slugUrl);
  }
}

// convert normal text into slug format
const convertUrlToSlug = (urlText) => {
  // Convert the URL text to lowercase
  let slug = urlText.toLowerCase();

  // Replace any non-alphanumeric characters with dashes
  slug = slug.replace(/[^a-z0-9]/g, '-');

  // Remove consecutive dashes
  slug = slug.replace(/-{2,}/g, '-');

  // Remove dashes from the beginning and end of the slug
  slug = slug.replace(/^-+|-+$/g, '');

  return slug;
}
</script>

<template>
  <h4 class="text-h4 mb-4">Integration</h4>
  <VRow class="match-height">
    <VCol
      cols="12"
      md="4"
      v-for="integration in integrationData" :key="integration.id"
      class="px-5 mb-4"
    >
      <VCard 
        class="integration-card" 
        :style="{ 
            backgroundImage: 'url(' + (integration.isHovered ? starConnectionWhiteIcon : starConnectionIcon) + ')' 
        }"
        @mouseover="handleMouseOver(integration)"
        @mouseleave="handleMouseLeave(integration)"
        :class="{'bg-primary': integration.isHovered}"
        @click="gotoSinglePage(integration.title)"
        >
        <VCardText class="d-flex gap-y-2 flex-column">
          <VImg
              :src="integration.isHovered ? integration.hoverImage : integration.image"
              :alt="integration.title"
              :width="90"
              :height="90"
              class="mb-2"
              />
          <h2 class="font-weight-medium px-1 mb-1" :class="{'text-primary': !integration.isHovered, 'text-white': integration.isHovered}">
            {{ integration.title }}
          </h2>
          <div class="text-base px-1">
            <p class="mb-0 text-base">
              {{ integration.description }}
            </p>
          </div>
        </VCardText>
      </VCard>
    </VCol>
  </VRow>
</template>
<style lang="scss" scoped>
.integration-card{
    cursor: pointer;
    background-position: right;
    background-size: contain;
    background-size: 264px;
    background-repeat: no-repeat;
    background-position-y: top;
}
</style>
