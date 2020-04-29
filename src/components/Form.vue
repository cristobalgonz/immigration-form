=<template>
  <div>
    <div class="container" v-if="english">
      <b-jumbotron header="Immigration Resource Form" lead="Last Updated: 04/28/2020" class="mt-3">
        <hr class="my-4">
        <p>This short form is meant as a means to help determine what immigration services you might qualify for and direct you towards resources. This, by no means, is a replacement for an attorney nor should it be regarded as legal advice, rather it is simply an informational tool.</p>
        <p><b>Note:</b> This form is completely safe and anonymous, none of your responses are saved or shared.</p>
        <div class="pt-5">
          <b-button variant="outline-primary" pill class="mr-2" :pressed="english" @click="eToggle">English</b-button>
          <b-button variant="outline-success" pill :pressed="spanish" @click="sToggle">Español</b-button>
        </div>
      </b-jumbotron>
      <b-form class="mb-3 bg-light px-5 pb-2 rounded" @submit="onSubmit" @reset="onReset" v-if="show">
        <!-- TODO: Ask about special skills, assets and add that piece to overall like "attractiveness" on results-->
        <!-- TODO: Add translation  -->
        <h5 class="pt-3 text-muted mb-0">Personal Questions <b-icon id="whyPersonalQ" icon="info-circle"></b-icon></h5>
        <b-popover target="whyPersonalQ" triggers="hover" placement="rightbottom">
          <template v-slot:title>Why these questions?</template>
          US Citizenship and Immigration Services recently announced new standards by which they will judge applicants seeking a green card through a family member, where before these types of questions were mostly asked of sponsors. Most of these factors aren't automatic disqualifiers, but we are trying to get a sense of what would be the strengths and weaknesses of your application.
        </b-popover>
        <b-form-group
          label-cols-lg="auto"
          label="How old are you?"
          label-size=""
          label-for="age"
          label-class="pt-5"
          class = "mb-0"
          >
          <b-form-input id="age" type="number" v-model="age" debounce="1000"></b-form-input>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Have you been abandoned, abused, or neglected by a parent?"
          label-size=""
          v-if="age && age < 22"
          label-for="neglected"
          label-class="pt-3"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="neglected" v-model="neglected" :options="yesNo" size=""></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="How large is your household?"
          label-size=""
          label-for="household"
          label-class="pt-3"
          class = "mb-0"
          >
          <b-form-input id="household" type="range" v-model="household" min=1 max=8 ></b-form-input>
          <p class="mb-0">Household size: {{household}}</p>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="What best describes your annual income level?"
          label-size=""
          label-class="pt-3"
          label-for="incomeLevel"
          class = "mb-0"
          >
          <b-form-radio-group id="incomeLevel" size="" v-model="incomeLevel">
            <b-form-radio value="low">Below ${{povertyLevels[household].low}}</b-form-radio>
            <b-form-radio value="mid">Between ${{povertyLevels[household].low}} and ${{povertyLevels[household].high}}</b-form-radio>
            <b-form-radio value="high">Above ${{povertyLevels[household].high}}</b-form-radio>
          </b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Have you ever been arrested?"
          label-size=""
          label-class="pt-3"
          label-for="arrested"
          class = "mb-0"
          >
          <b-form-radio-group id="arrested" size="" v-model="arrested" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Do you have or could you reasonably obtain health insurance in the US?"
          label-size=""
          label-class="pt-3"
          label-for="insured"
          class = "mb-0"
          >
          <b-form-radio-group id="insured" size="" v-model="insured" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="What is your highest level of education?"
          label-for="edLevel"
          label-class="pt-5"
          class="mb-0"
          >
          <b-form-select id="edLevel" v-model="edLevel" :options="edLevels">
            <template v-slot:first>
              <b-form-select-option :value="null" disabled>-- Please select an option --</b-form-select-option>
            </template>
          </b-form-select>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Do you have any specialized skills, certifications, or professional licenses?"
          label-size=""
          label-class="pt-3"
          label-for="specialSkills"
          class = "mb-0"
          >
          <b-form-radio-group id="specialSkills" size="" v-model="specialSkills" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <h5 class="pt-3 text-muted mb-0">Immigration History <b-icon id="whyImmigrationQ" icon="info-circle"></b-icon></h5>
        <b-popover target="whyImmigrationQ" triggers="hover" placement="right">
          <template v-slot:title>Why these questions?</template>
          Your past immigration history can affect current applications. For instance, if you stay in the US unlawfully for too long (known as unlawful presence), you can be barred from re-entering the US for some amount of time the moment you leave.
        </b-popover>

        <b-form-group
          label-cols-lg="auto"
          label="Have you every unlawfully entered the US or overstayed a visa?"
          label-size=""
          label-for="unlawfulEntry"
          label-class="pt-3"
          class = "mb-0"
          >
          <b-form-radio-group id="unlawfulEntry" v-model="unlawfulEntry" :options="yesNo" size=""></b-form-radio-group>
        </b-form-group>

        <b-form-group
          v-if="unlawfulEntry"
          label-cols-lg="auto"
          label="Did you stay in the US illegally for more than 180 days?"
          label-size=""
          label-for="threeBar"
          label-class="pt-3"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="threeBar" v-model="threeBar" :options="yesNo" size=""></b-form-radio-group>
        </b-form-group>

        <b-form-group
          v-if="threeBar"
          label-cols-lg="auto"
          label="Did you stay in the US illegally for more than a year?"
          label-size=""
          label-for="tenBar"
          label-class="pt-3"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="tenBar" v-model="tenBar" :options="yesNo" size=""></b-form-radio-group>
        </b-form-group>

        <b-form-group
          v-if="tenBar"
          label-cols-lg="auto"
          label="Did you leave the US and reenter after spending more than a year here illegally?"
          label-size=""
          label-for="permBar"
          label-class="pt-3"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="permBar" v-model="permBar" :options="yesNo" size=""></b-form-radio-group>
        </b-form-group>

        <b-form-group
          v-if="unlawfulEntry"
          label-cols-lg="auto"
          label="Have you been detained in any entry or exit of the US or deported?"
          label-size=""
          label-for="detained"
          label-class="pt-3"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="detained" v-model="detained" :options="yesNo" size=""></b-form-radio-group>
        </b-form-group>

        <h5 class="pt-3 text-muted mb-0">Family Questions <b-icon id="whyFamQ" icon="info-circle"></b-icon></h5>
        <b-popover target="whyFamQ" triggers="hover" placement="right">
          <template v-slot:title>Why these questions?</template>
          These questions help inform who in your family might be able to sponsor your petition for lawful residence in the United States.
        </b-popover>

        <b-form-group
          label-cols-lg="auto"
          label="Are your parents US citizens?"
          label-size=""
          label-class="pt-3"
          label-for="parentUS"
          class = "mb-0"
          >
          <b-form-radio-group id="parentUS" size="" v-model="parentUS" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="If your parents aren't US citizens, are they Lawful Permanent Residents (Green Card holders)?"
          label-size=""
          label-class="pt-3"
          label-for="parentLPR"
          class = "mb-0"
          >
          <b-form-radio-group id="parentLPR" size="" v-model="parentLPR" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <!-- Marriage -->
        <b-form-group
          label-cols-lg="auto"
          label="Are you married?"
          label-size=""
          label-class="pt-3"
          label-for="married"
          class = "mb-0"
          >
          <b-form-radio-group id="married" class="" v-model="married" :options="yesNo" size=""></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          v-if="married"
          label="What is your spouse's immigration status?"
          label-for="spouseStatus"
          label-class="pt-5"
          class="ml-5 mb-0"
          >
          <b-form-select id="spouseStatus" v-model="spouseStatus" :options="status">
            <template v-slot:first>
              <b-form-select-option :value="null" disabled>-- Please select an option --</b-form-select-option>
            </template>
          </b-form-select>
        </b-form-group>

        <!-- children -->
        <b-form-group
          label-cols-lg="auto"
          label="Do you have children?"
          label-size=""
          label-class="pt-3"
          label-for="children"
          class = "mb-0"
          >
          <b-form-radio-group id="children" v-model="children" :options="yesNo" size=""></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          v-if="children"
          label="Are any of your children US citizens?"
          label-for="childUSC"
          label-class="pt-3"
          class="ml-5 mb-0"
          >
          <b-form-radio-group id="childUSC" v-model="childUSC" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Do you have any siblings that are US citizens?"
          label-for="sibUSC"
          label-class="pt-3"
          class=" mb-0"
          >
          <b-form-radio-group id="sibUSC" v-model="sibUSC" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <h5 class="pt-3 text-muted mb-0">Other <b-icon id="whyOtherQ" icon="info-circle"></b-icon></h5>
        <b-popover target="whyOtherQ" triggers="hover" placement="right">
          <template v-slot:title>Why these questions?</template>
          These questions largely pertain to experiences that might qualify you for a particular visa, status, or program.
        </b-popover>
        <b-form-group
          label-cols-lg="auto"
          label="Do you have any fear of returning to your country?"
          label-size=""
          label-class="pt-3"
          label-for="wantAsylum"
          class = "mb-0"
          >
          <b-form-radio-group id="wantAsylum" size="" v-model="wantAsylum" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          v-if="wantAsylum"
          label="Is this due to persecution or fear of persecution due to your race, religion, nationality, political opinion, or membership in a particular social group?"
          label-size=""
          label-class="pt-3 pb-0"
          label-for="warrantAsylum"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="warrantAsylum" v-model="warrantAsylum" :options="yesNo"></b-form-radio-group>
        </b-form-group>
        <!-- dropdown for reasons why -->

        <b-form-group
          label-cols-lg="auto"
          label="Have you ever been a victim of domestic abuse by a spouse, parent, or child while in the US?"
          label-size=""
          label-class="pt-3"
          label-for="domAbuse"
          class = "mb-0"
          >
          <b-form-radio-group id="domAbuse" size="" v-model="domAbuse" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Was that person a US citizen or Lawful Permanent Resident?"
          v-if="domAbuse"
          label-size=""
          label-class="pt-3"
          label-for="abuserUSC"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="abuserUSC" size="" v-model="abuserUSC" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Have you ever been threatened or harmed by a spouse, parent, or child while in the US?"
          label-size=""
          label-class="pt-3"
          label-for="domThreat"
          class = "mb-0"
          >
          <b-form-radio-group id="domThreat" size="" v-model="domThreat" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Was that person a US citizen or Lawful Permanent Resident?"
          v-if="domThreat"
          label-size=""
          label-class="pt-3"
          label-for="threatenerUSC"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="threatenerUSC" size="" v-model="threatenerUSC" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Have you ever been a victim of a crime in the US?"
          label-size=""
          label-class="pt-3"
          label-for="vicCrime"
          class = "mb-0"
          >
          <b-form-radio-group id="vicCrime" size="" v-model="vicCrime" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          v-if="vicCrime"
          label-cols-lg="auto"
          label="Did you report it to the police or help the investigation or prosecution?"
          label-size=""
          label-class="pt-3"
          label-for="reportedCrime"
          class = "mb-0 ml-5"
          >
          <b-form-radio-group id="reportedCrime" size="" v-model="reportedCrime" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Did anyone recruit you in your home country to work in the United States?"
          label-size=""
          label-class="pt-3"
          label-for="recruitWork"
          class = "mb-0"
          >
          <b-form-radio-group id="recruitWork" size="" v-model="recruitWork" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Did you feel forced to work or tricked to work?"
          label-size=""
          label-class="pt-3"
          label-for="trickedToWork"
          class = "mb-0"
          >
          <b-form-radio-group id="trickedToWork" size="" v-model="trickedToWork" :options="yesNo"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="Were you required to work without pay? Or less pay than allowed or expected?"
          label-size=""
          label-class="pt-3"
          label-for="exploited"
          >
          <b-form-radio-group id="exploited" v-model="exploited" :options="yesNo" size=""></b-form-radio-group>
        </b-form-group>

        <b-button pill type="submit" class="mr-2" variant="primary">Submit</b-button>
        <b-button pill type="reset" variant="danger">Reset</b-button>
      </b-form>

      <b-row>
        <b-col sm="8">
          <b-card class="mt-3 rounded" header="Results">
            <b-list-group v-if="results.length !== 0">
              <b-list-group-item v-for="result in results" :key="result.eng" >
                {{ result.eng }}
              </b-list-group-item>
            </b-list-group>
          </b-card>
        </b-col>
        <b-col sm="4">
          <b-card class="mt-3 mb-2 rounded" header="Resources">
            <b-list-group>
              <b-list-group-item v-if="waiver" href="https://www.uscis.gov/family/family-us-citizens/provisional-unlawful-presence-waivers" target="_blank">Unlawful Presence Waivers</b-list-group-item>
              <b-list-group-item v-if="preq" href="https://www.uscis.gov/family/family-us-citizens/bringing-children-sons-and-daughters-live-united-states-permanent-residents" target="_blank">Sponsor: Parent</b-list-group-item>
              <b-list-group-item v-if="sreq" href="https://www.uscis.gov/family/family-us-citizens/bringing-spouses-live-united-states-permanent-residents" target="_blank">Sponsor: Spouse</b-list-group-item>
              <b-list-group-item v-if="childreq" href="https://www.uscis.gov/family/family-us-citizens/bringing-parents-live-united-states-permanent-residents" target="_blank">Sponsor: Child</b-list-group-item>
              <b-list-group-item v-if="sibreq" href="https://www.uscis.gov/family/family-us-citizens/bringing-siblings-live-united-states-permanent-residents" target="_blank">Sponsor: Sibling</b-list-group-item>
              <b-list-group-item v-if="asylum" href="https://www.uscis.gov/humanitarian/refugees-and-asylum/asylum/obtaining-asylum-united-states" target="_blank">Seeking Asylum</b-list-group-item>
              <b-list-group-item v-if="vawa" href="https://www.uscis.gov/humanitarian/battered-spouse-children-and-parents" target="_blank">Violence Against Women's Act</b-list-group-item>
              <b-list-group-item v-if="uvisa" href="https://www.uscis.gov/humanitarian/victims-human-trafficking-and-other-crimes/victims-criminal-activity-u-nonimmigrant-status" target="_blank">U-visa</b-list-group-item>
              <b-list-group-item v-if="tvisa" href="https://www.uscis.gov/humanitarian/victims-human-trafficking-and-other-crimes/victims-human-trafficking-t-nonimmigrant-status" target="_blank">T-visa</b-list-group-item>
              <b-list-group-item v-if="sijs" href="https://www.uscis.gov/working-united-states/sij" target="_blank">Special Immigrant Juvenile Status</b-list-group-item>
            </b-list-group>
          </b-card>
        </b-col>
      </b-row>

      <b-card id="Public Charge" class="mb-4 rounded">
        <template v-slot:header>
          <div class="mb-0">Public Charge Status <b-icon id="whyPublicCharge" icon="info-circle"></b-icon></div>
        </template>
        <b-popover target="whyPublicCharge" triggers="hover" placement="top">
          <template v-slot:title>What does this mean?</template>
          The following lists display potential strengths and weaknesses in your application that relate to being seen as a "public charge," or in other words, a burden on society.
        </b-popover>
        <b-row v-if="strengths.length !== 0 || weaknesses.length !== 0">
          <b-col sm="6">
            <h6 class="text-center">Strengths</h6>
            <b-list-group>
              <b-list-group-item v-for="strength in strengths" :key="strength.eng" >
                {{ strength.eng }}
              </b-list-group-item>
            </b-list-group>
          </b-col>
          <b-col sm="6">
            <h6 class="text-center">Weaknesses</h6>
            <b-list-group>
              <b-list-group-item v-for="weakness in weaknesses" :key="weakness.eng" >
                {{ weakness.eng }}
              </b-list-group-item>
            </b-list-group>
          </b-col>
        </b-row>
      </b-card>
    </div>
    <div class="container" v-if="spanish">
      <b-jumbotron header="Formulario de Recursos de Inmigración" lead="Última actualización: 04/28/2020" class="mt-3">
        <hr class="my-4">
        <p>Este breve formulario le ayudará determinar para qué servicios de inmigración podría calificar y recursos disponibles. Esto, de ninguna manera, reemplaza un abogado de inmigración ni debe considerarse como asesoramiento legal, sino que es simplemente una herramienta informativa. </p>
        <p><b>Nota:</b> Este formulario es completamente confidencial y anónimo, ninguna de sus respuestas se graba o comparte.</p>
        <div class="pt-5">
          <b-button variant="outline-primary" pill class="mr-2" :pressed="english" @click="eToggle">English</b-button>
          <b-button variant="outline-success" pill :pressed="spanish" @click="sToggle">Español</b-button>
        </div>
      </b-jumbotron>
      <b-form class="mb-3 bg-light px-5 pb-2 rounded" @submit="onSubmit" @reset="onReset" v-if="show">
        <!-- TODO: Ask about special skills, assets and add that piece to overall like "attractiveness" on results-->
        <!-- TODO: Add translation  -->
        <h5 class="pt-3 text-muted mb-0">Datos personales <b-icon id="whyPersonalQ" icon="info-circle"></b-icon></h5>
        <b-popover target="whyPersonalQ" triggers="hover" placement="rightbottom">
          <template v-slot:title>¿Por qué estas preguntas?</template>
          US Citizenship and Immigration Services anunció recientemente nuevos estándares por los cuales juzgarán a los solicitantes de Green Cards a través de un miembro de la familia, donde antes solo se hacía este tipo de preguntas a los patrocinadores. La mayoría de estos factores no te van a descalificar automáticamente, pero estamos tratando de formar una idea de cuáles serían las fortalezas y debilidades de su aplicación.
        </b-popover>
        <b-form-group
          label-cols-lg="auto"
          label="¿Cuántos años tiene?"
          label-size=""
          label-for="age"
          label-class="pt-5"
          class = "mb-0"
          >
          <b-form-input id="age" type="number" v-model="age" debounce="1000"></b-form-input>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Ha sido abandonado, abusado o maltratado por un padre?"
          label-size=""
          v-if="age && age < 22"
          label-for="neglected"
          label-class="pt-3"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="neglected" v-model="neglected" :options="yesNo" text-field="text_es" size=""></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Cuántas personas dependen de usted economicamente (hijos, cónyuge, etc.)?"
          label-size=""
          label-for="household"
          label-class="pt-3"
          class = "mb-0"
          >
          <b-form-input id="household" type="range" v-model="household" min=1 max=8 ></b-form-input>
          <p class="mb-0">Numero de personas: {{household}}</p>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Cual es su nivel de ingreso anual?"
          label-size=""
          label-class="pt-3"
          label-for="incomeLevel"
          class = "mb-0"
          >
          <b-form-radio-group id="incomeLevel" size="" v-model="incomeLevel">
            <b-form-radio value="low">Menos de ${{povertyLevels[household].low}}</b-form-radio>
            <b-form-radio value="mid">Entre ${{povertyLevels[household].low}} y ${{povertyLevels[household].high}}</b-form-radio>
            <b-form-radio value="high">Más de ${{povertyLevels[household].high}}</b-form-radio>
          </b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Alguna vez ha sido arrestado o detinido por las autoridades en los EE.UU.?"
          label-size=""
          label-class="pt-3"
          label-for="arrested"
          class = "mb-0"
          >
          <b-form-radio-group id="arrested" size="" v-model="arrested" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Tiene o podría obtener seguro de salud en los EE.UU.?"
          label-size=""
          label-class="pt-3"
          label-for="insured"
          class = "mb-0"
          >
          <b-form-radio-group id="insured" size="" v-model="insured" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Cual es tu nivel de educación?"
          label-for="edLevel"
          label-class="pt-5"
          class="mb-0"
          >
          <b-form-select id="edLevel" v-model="edLevel" :options="edLevels" text-field="text_es">
            <template v-slot:first>
              <b-form-select-option :value="null" disabled>-- Por favor eliga una opción --</b-form-select-option>
            </template>
          </b-form-select>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Tiene alguna especialización, certificación o licencia profesional?"
          label-size=""
          label-class="pt-3"
          label-for="specialSkills"
          class = "mb-0"
          >
          <b-form-radio-group id="specialSkills" size="" v-model="specialSkills" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <h5 class="pt-3 text-muted mb-0">Historial de inmigración <b-icon id="whyImmigrationQ" icon="info-circle"></b-icon></h5>
        <b-popover target="whyImmigrationQ" triggers="hover" placement="right">
          <template v-slot:title>¿Por qué estas preguntas?</template>
          Su historial de inmigración puede afectar las solicitudes actuales. Por ejemplo, si permanece en los EE.UU. ilegalmente durante demasiado tiempo (lo que se conoce como "unlawful presence"), se le puede prohibir el reingreso a los EE.UU. durante cierto tiempo si tengan registro de tu salida o entrada.
        </b-popover>
        <b-form-group
          label-cols-lg="auto"
          label="¿Alguna vez ingresó ilegalmente a los Estados Unidos o excedió una visa?"
          label-size=""
          label-for="unlawfulEntry"
          label-class="pt-3"
          class = "mb-0"
          >
          <b-form-radio-group id="unlawfulEntry" v-model="unlawfulEntry" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          v-if="unlawfulEntry"
          label-cols-lg="auto"
          label="¿Permaneció ilegalmente en los Estados Unidos por más de 180 días?"
          label-size=""
          label-for="threeBar"
          label-class="pt-3"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="threeBar" v-model="threeBar" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          v-if="threeBar"
          label-cols-lg="auto"
          label="¿Permaneció ilegalmente en los Estados Unidos por más de un año?"
          label-size=""
          label-for="tenBar"
          label-class="pt-3"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="tenBar" v-model="tenBar" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          v-if="tenBar"
          label-cols-lg="auto"
          label="¿Salió de los EE.UU. y volvió a entrar después de pasar más de un año aquí ilegalmente?"
          label-size=""
          label-for="permBar"
          label-class="pt-3"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="permBar" v-model="permBar" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          v-if="unlawfulEntry"
          label-cols-lg="auto"
          label="¿Ha sido detenido en alguna entrada o salida de los Estados Unidos o deportado?"
          label-size=""
          label-for="detained"
          label-class="pt-3"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="detained" v-model="detained" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <h5 class="pt-3 text-muted mb-0">Preguntas sobre su familia <b-icon id="whyFamQ" icon="info-circle"></b-icon></h5>
        <b-popover target="whyFamQ" triggers="hover" placement="right">
          <template v-slot:title>¿Por qué estas preguntas?</template>
          Estas preguntas ayudan a informar quién en su familia podría patrocinar su petición de residencia legal en los Estados Unidos.
        </b-popover>

        <b-form-group
          label-cols-lg="auto"
          label="¿Son ciudadanos americanos sus padres?"
          label-size=""
          label-class="pt-3"
          label-for="parentUS"
          class = "mb-0"
          >
          <b-form-radio-group id="parentUS" size="" v-model="parentUS" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Si no son ciudadanos, son sus padres residentes legales (Green Card)?"
          label-size=""
          label-class="pt-3"
          label-for="parentLPR"
          class = "mb-0"
          >
          <b-form-radio-group id="parentLPR" size="" v-model="parentLPR" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <!-- Marriage -->
        <b-form-group
          label-cols-lg="auto"
          label="¿Estas casado(a)?"
          label-size=""
          label-class="pt-3"
          label-for="married"
          class = "mb-0"
          >
          <b-form-radio-group id="married" class="" v-model="married" :options="yesNo" text-field="text_es" size=""></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          v-if="married"
          label="¿Cuál es el estado migratorio de su cónyuge?"
          label-for="spouseStatus"
          label-class="pt-5"
          class="ml-5 mb-0"
          >
          <b-form-select id="spouseStatus" v-model="spouseStatus" :options="status" text-field="text_es">
            <template v-slot:first>
              <b-form-select-option :value="null" disabled>-- Por favor eliga una opción --</b-form-select-option>
            </template>
          </b-form-select>
        </b-form-group>

        <!-- children -->
        <b-form-group
          label-cols-lg="auto"
          label="¿Tiene hijos?"
          label-size=""
          label-class="pt-3"
          label-for="children"
          class = "mb-0"
          >
          <b-form-radio-group id="children" v-model="children" :options="yesNo" text-field="text_es" size=""></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          v-if="children"
          label="¿Alguno de sus hijos es ciudadano americano?"
          label-for="childUSC"
          label-class="pt-3"
          class="ml-5 mb-0"
          >
          <b-form-radio-group id="childUSC" v-model="childUSC" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Tiene hermanos que sean ciudadanos americanos?"
          label-for="sibUSC"
          label-class="pt-3"
          class=" mb-0"
          >
          <b-form-radio-group id="sibUSC" v-model="sibUSC" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <h5 class="pt-3 text-muted mb-0">Preguntas adicionales <b-icon id="whyOtherQ" icon="info-circle"></b-icon></h5>
        <b-popover target="whyOtherQ" triggers="hover" placement="right">
          <template v-slot:title>¿Por qué estas preguntas?</template>
          Estas preguntas se refieren en gran medida a experiencias que podrían calificarlo para una visa, ayuda o programa en particular.
        </b-popover>
        <b-form-group
          label-cols-lg="auto"
          label="¿Tiene miedo de volver a su país?"
          label-size=""
          label-class="pt-3"
          label-for="wantAsylum"
          class = "mb-0"
          >
          <b-form-radio-group id="wantAsylum" size="" v-model="wantAsylum" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>
        <b-form-group
          label-cols-lg="auto"
          v-if="wantAsylum"
          label="¿Se debe a la persecución o al miedo a la persecución debido a su raza, religión, nacionalidad, opinión política o pertenencia a un grupo social en particular?"
          label-size=""
          label-class="pt-3 pb-0"
          label-for="warrantAsylum"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="warrantAsylum" size="" v-model="warrantAsylum" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Alguna vez ha sido víctima de abuso doméstico por parte de un cónyuge, padre o hijo en los EE.UU.?"
          label-size=""
          label-class="pt-3"
          label-for="domAbuse"
          class = "mb-0"
          >
          <b-form-radio-group id="domAbuse" size="" v-model="domAbuse" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Era esa persona un ciudadano americano o residente legal?"
          v-if="domAbuse"
          label-size=""
          label-class="pt-3"
          label-for="abuserUSC"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="abuserUSC" size="" v-model="abuserUSC" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Alguna vez ha sido amenazado o perjudicado por un cónyuge, padre o hijo en los EE.UU.?"
          label-size=""
          label-class="pt-3"
          label-for="domThreat"
          class = "mb-0"
          >
          <b-form-radio-group id="domThreat" size="" v-model="domThreat" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Era esa persona un ciudadano americano o residente legal?"
          v-if="domThreat"
          label-size=""
          label-class="pt-3"
          label-for="threatenerUSC"
          class = "ml-5 mb-0"
          >
          <b-form-radio-group id="threatenerUSC" size="" v-model="threatenerUSC" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Alguna vez ha sido víctima de un crimen en los EE.UU.?"
          label-size=""
          label-class="pt-3"
          label-for="vicCrime"
          class = "mb-0"
          >
          <b-form-radio-group id="vicCrime" size="" v-model="vicCrime" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          v-if="vicCrime"
          label-cols-lg="auto"
          label="¿Lo denunció a la policía o ayudó en la investigación o el enjuiciamiento?"
          label-size=""
          label-class="pt-3"
          label-for="reportedCrime"
          class = "mb-0 ml-5"
          >
          <b-form-radio-group id="reportedCrime" size="" v-model="reportedCrime" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Alguien lo contrató en su país de origen para trabajar en los Estados Unidos?"
          label-size=""
          label-class="pt-3"
          label-for="recruitWork"
          class = "mb-0"
          >
          <b-form-radio-group id="recruitWork" size="" v-model="recruitWork" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Te sentiste obligado a trabajar o engañado para trabajar?"
          label-size=""
          label-class="pt-3"
          label-for="trickedToWork"
          class = "mb-0"
          >
          <b-form-radio-group id="trickedToWork" size="" v-model="trickedToWork" :options="yesNo" text-field="text_es"></b-form-radio-group>
        </b-form-group>

        <b-form-group
          label-cols-lg="auto"
          label="¿Tuviste que trabajar sin paga? ¿O menos paga de lo permitido o esperado?"
          label-size=""
          label-class="pt-3"
          label-for="exploited"
          >
          <b-form-radio-group id="exploited" v-model="exploited" :options="yesNo" text-field="text_es" size=""></b-form-radio-group>
        </b-form-group>

        <b-button pill type="submit" class="mr-2" variant="primary">Resultados</b-button>
        <b-button pill type="reset" variant="danger">Reiniciar</b-button>
      </b-form>

      <b-row>
        <b-col sm="8">
          <b-card class="mt-3 rounded" header="Resultados">
            <b-list-group v-if="results.length !== 0">
              <b-list-group-item v-for="result in results" :key="result.esp" >
                {{ result.esp }}
              </b-list-group-item>
            </b-list-group>
          </b-card>
        </b-col>
        <b-col sm="4">
          <b-card class="mt-3 mb-2 rounded" header="Recursos">
            <b-list-group>
              <b-list-group-item v-if="waiver" href="https://www.uscis.gov/es/exencionprovisional" target="_blank">Exención Provisional por Presencia Ilegal</b-list-group-item>
              <b-list-group-item v-if="preq" href="https://www.uscis.gov/es/familia/familiares-de-ciudadanos-estadounidenses/para-hijos-o-hijas-menores-de-edad-a-vivir-en-los-ee-uu-como-residentes-permanentes" target="_blank">Patrocinador: padre</b-list-group-item>
              <b-list-group-item v-if="sreq" href="https://www.uscis.gov/es/familia/familiares-de-ciudadanos-estadounidenses/conyuge" target="_blank">Patrocinador: cónyuge</b-list-group-item>
              <b-list-group-item v-if="childreq" href="https://www.uscis.gov/es/familia/familiares-de-ciudadanos-estadounidenses/padres-tarjeta-verde/para-traer-a-los-padres-a-vivir-en-los-estados-unidos-como-residentes-permanentes" target="_blank">Patrocinador: hijo</b-list-group-item>
              <b-list-group-item v-if="sibreq" href="https://www.uscis.gov/es/familia/familiares-de-ciudadanos-estadounidenses/como-traer-hermanos-a-vivir-en-los-ee-uu-como-residentes-permanentes" target="_blank">Patrocinador: hermano</b-list-group-item>
              <b-list-group-item v-if="asylum" href="https://www.uscis.gov/es/programas-humanitarios/obtener-asilo-en-los-estados-unidos" target="_blank">Asilo</b-list-group-item>
              <b-list-group-item v-if="vawa" href="https://www.uscis.gov/es/programas-humanitarios/conyuge-hijos-y-padres-abusados" target="_blank">Cónyuge, hijos y padres abusados (VAWA)</b-list-group-item>
              <b-list-group-item v-if="uvisa" href="https://www.uscis.gov/es/programas-humanitarios/victimas-de-la-trata-de-personas-y-de-otros-crimenes/victimas-de-actos-criminales-estatus-u-de-no-inmigrante" target="_blank">Visa U</b-list-group-item>
              <b-list-group-item v-if="tvisa" href="https://www.uscis.gov/es/programas-humanitarios/victimas-de-la-trata-de-personas-y-de-otros-crimenes/victimas-de-trata-de-personas-estatus-t-de-no-inmigrante" target="_blank">Visa T</b-list-group-item>
              <b-list-group-item v-if="sijs" href="https://www.uscis.gov/es/trabajar-en-los-estados-unidos/sij" target="_blank">Jóvenes Inmigrantes Especiales</b-list-group-item>
            </b-list-group>
          </b-card>
        </b-col>
      </b-row>

      <b-card id="Public Charge" class="mb-4 rounded">
        <template v-slot:header>
          <div class="mb-0">Estado de carga pública <b-icon id="whyPublicCharge" icon="info-circle"></b-icon></div>
        </template>
        <b-popover target="whyPublicCharge" triggers="hover" placement="top">
          <template v-slot:title>¿Qué significa esto?</template>
          Las siguientes listas muestran las fortalezas y debilidades potenciales en su aplicación que se relacionan con ser visto como una "carga pública."
        </b-popover>
        <b-row v-if="strengths.length !== 0 || weaknesses.length !== 0">
          <b-col sm="6">
            <h6 class="text-center">Fortalezas</h6>
            <b-list-group>
              <b-list-group-item v-for="strength in strengths" :key="strength.esp" >
                {{ strength.esp }}
              </b-list-group-item>
            </b-list-group>
          </b-col>
          <b-col sm="6">
            <h6 class="text-center">Debilidades</h6>
            <b-list-group>
              <b-list-group-item v-for="weakness in weaknesses" :key="weakness.esp" >
                {{ weakness.esp }}
              </b-list-group-item>
            </b-list-group>
          </b-col>
        </b-row>
      </b-card>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        //countryOfOrigin: '',
        english: true,
        spanish: false,
        age: '',
        household: 4,
        incomeLevel: '',
        insured: null,
        edLevel: '',
        specialSkills: null,
        unlawfulEntry: null,
        threeBar: null,
        tenBar: null,
        permBar: null,
        detained: null,
        parentUS: null,
        married: null,
        spouseStatus: '',
        children: null,
        childUSC: null,
        sibUSC: null,
        parentLPR: null,
        arrested: null,
        wantAsylum: null,
        warrantAsylum: null,
        domAbuse: null,
        abuserUSC: null,
        threatenerUSC: null,
        domThreat: null,
        vicCrime: null,
        reportedCrime: null,
        recruitWork: null,
        trickedToWork: null,
        exploited: null,
        neglected: null,
        show: true,

        vawa: false,
        uvisa: false,
        tvisa: false,
        sijs: false,
        preq: false,
        sreq: false,
        sibreq: false,
        childreq: false,
        asylum: false,
        waiver: false,
        results: [],
        strengths: [],
        weaknesses: [],

        yesNo: [{value: true, text: 'Yes', text_es:'Sí'}, {value: false, text: 'No', text_es: 'No'}],

        status:[{value:'USC', text: 'US citizen', text_es: 'Ciudadano americano'}, {value:'LPR', text:'Lawful permanent resident', text_es: 'Residente legal'}, {value:'Visa', text:'Visa holder', text_es:'Visa'}, {value:'Undocumented', text:'Undocumented', text_es:'Indocumentado'}],

        edLevels: [{value:'low', text: 'No high school degree', text_es: 'Sin título de secundaria'}, {value:'highschool', text:'High school degree or GED', text_es: 'Título de secundaria o GED'}, {value:'bachelors', text:'Bachelors Degree', text_es:'Licenciatura'}, {value:'graduate', text:'Graduate or Professional Degree', text_es: 'Posgrado o doctorado'}],

        povertyLevels: [{low: 0, high: 0}, {low: 19150, high: 25520}, {low: 25860, high: 34480}, {low: 32580, high: 43440}, {low: 39300, high: 52400}, {low: 46020, high: 61360}, {low: 52740, high: 70320}, {low: 59460, high: 79280}, {low: 66180, high: 88240}],

      }
    },
    methods: {
      eToggle(){
        this.english = true
        this.spanish = false
      },
      sToggle(){
        this.english = false
        this.spanish = true
      },
      onSubmit(evt) {
        evt.preventDefault()

        if(this.age > 18 && this.age < 62){
          this.strengths.push({eng: "You are within the ages of 18 to 61, showing you are old enough to work and not at retirement age.", esp:"Tiene entre 18 y 61 años, lo que demuestra que tiene la edad suficiente para trabajar y no de edad de jubilación."})
        }
        else{
          this.weaknesses.push({eng: "You are not within the age range of 18 to 61.", esp: "No está dentro del rango de edad de 18 a 61."})
        }
        if(this.incomeLevel === "low"){
          this.weaknesses.push({eng: "Your income level considering your household size is low.", esp: "Su nivel de ingresos teniendo en cuenta el tamaño de su hogar es bajo."})
        }
        else if(this.incomeLevel === "mid"){
          this.strengths.push({eng: "Your income level is an acceptable range for your household size.", esp: "Su nivel de ingresos es un rango aceptable para el tamaño de su hogar."})
        }
        else if(this.incomeLevel === "high"){
          this.strengths.push({eng: "Your income level is a positive for your application because it is twice as much as the poverty level for your household size.", esp: "Su nivel de ingresos es positivo para su solicitud porque es el doble que el nivel oficial de pobreza para el tamaño de su hogar."})
        }
        if(this.insured){
          this.strengths.push({eng: "Positive if you can prove that you have or can get health insurance.", esp: "Positivo si puede demostrar que tiene o puede obtener un seguro de salud."})
        }
        else{
          this.weaknesses.push({eng: "Not having or being unable to get health insurance.", esp: "No tener o no poder obtener un seguro de salud."})
        }
        if(this.edLevel === "low"){
          this.weaknesses.push({eng: "Not having a high school degree or GED.", esp: "No tener un título de escuela secundaria o GED."})
        }
        else if(this.edLevel === "bachelors" || this.edLevel === "graduate"){
          this.strengths.push({eng: "Having above a high school level education, as it signals the ability to get a good job.", esp: "Tener una educación superior a la secundaria, porque indica la capacidad de conseguir un buen trabajo."})
        }
        if(this.specialSkills){
          this.strengths.push({eng: "Specialized skills, professional licenses, or certifications demonstrate that you can likely find good employment.", esp: "Las especializaciones, licencias profesionales o certificaciones demuestran que probablemente pueda encontrar un buen empleo."})
        }
        if(this.arrested){
          this.weaknesses.push({eng: "Prior arrests or criminal history can potentially be harmful in applications.", esp: "Los arrestos previos o antecedentes penales pueden ser potencialmente dañinos en las aplicaciones."})
        }

        if(this.parentUS){
          if (this.married){
            this.results.push({eng: "Parental Sponsorship: Since your parent is a US citizen, they can petition for you to become a permanent resident. Since you are married, the wait time is generally longer than for unmarried children of US citizens, but your spouse and children can be requested along with you. For more information, check Resources.", esp: "Patrocinio de los padres: dado que sus padres son ciudadanos americanos, usted podría calificar para la residencia. Como está casado, el tiempo de espera generalmente es más largo que para los hijos solteros de ciudadanos americanos, pero se puede solicitar a su cónyuge e hijos junto con usted. Para más información, ver Recursos."})
            this.preq = true
          }
          else{
            this.results.push({eng: "Parental Sponsorship: Since your parent is a US citizen, they can petition for you to become a permanent resident. Since you are unmarried, this tends to be a very fast process with low wait times. For more information, check Resources.", esp: "Patrocinio de los padres: dado que sus padres son ciudadanos americanos, usted podría calificar para la residencia. Como no estás casado, este suele ser un proceso muy rápido con bajos tiempos de espera. Para más información, ver Recursos."})
            this.preq = true
          }
        }
        else if(this.spouseStatus === "USC" || this.spouseStatus === "LPR"){
          this.results.push({eng: "Spousal Sponsorship: Since your spouse is a US Citizen or Lawful Permanent Resident, they can petition for you to become a permanent resident. For more information, check Resources.", esp: "Patrocinio conyugal: dado que su cónyuge es ciudadano americano o residente permanente legal, usted podría calificar para la residencia. Para más información, ver Recursos."})
          this.sreq = true
        }
        else if(this.childUSC){
          this.results.push({eng: "Child Sponsorship: Since your child is a US citizen, they can petition for you to become a permanent resident once they are 21. Note that it is very challenging to get a waiver for unlawful presence bans if your child is requesting you. For more information, check Resources.", esp: "Patrocinio de su hijo: dado que su hijo es ciudadano de los EE.UU., usted podría calificar para la residencia una vez que tenga 21 años tu hijo/a. Tenga en cuenta que es muy difícil obtener una exención por prohibiciones de presencia ilegal si su hijo lo solicita. Para más información, ver Recursos."})
          this.childreq = true
        }
        else if(this.sibUSC){
          this.results.push({eng: "Sibling Sponsorship: Since your sibling is a USC, they can petition for you to become a permanent resident. Note that these petitions are considered low priority by US Immigration Services, so might take longer than other petitions. For more information, check Resources.", esp: "Patrocinio de hermanos: dado que su hermano es ciudadano de los EE.UU., usted podría calificar para la residencia. Tenga en cuenta que estas peticiones son consideradas de baja prioridad por los Servicios de Inmigración de los EE.UU., por lo cual pueden llevar más tiempo que otras. Para más información, ver Recursos."})
          this.sibreq = true
        }
        else if(this.parentLPR && !this.married){
          this.results.push({eng: "Parental Sponsorship: Since your parent is a Lawful Permanent Resident (LPR) and you are unmarried, they can petition for you to become a permanent resident. For more information, check Resources.", esp: "Patrocinio de los padres: dado que su padre es residente legal, usted podría calificar para la residencia. Para más información, ver Recursos."})
          this.preq = true
        }

        if(this.warrantAsylum){
          this.results.push({eng: "Asylum: You might qualify for asylum or refugee protection due to the dangers you might face in your country of origin. For more information, check Resources.", esp: "Asilo: puede calificar para asilo o protección de refugiados debido a los peligros que podría enfrentar en su país de origen. Para más información, ver Recursos."})
          this.asylum = true
        }

        if(this.domAbuse || this.domThreat){
          if (this.abuserUSC || this.threatenerUSC){
            this.results.push({eng: "Domestic Violence Protection: Due to your history with domestic abuse or harm at the hands of a family member, you might be eligible for a U-visa or support through the Violence Against Women's Act (VAWA). For more information, check Resources.", esp: "Protección contra la violencia doméstica: debido a su historial de abuso o daño doméstico a manos de un miembro de la familia, puede ser elegible para una visa U o apoyo a través de la Ley de Violencia contra la Mujer (VAWA). Para más información, ver Recursos."})
            this.vawa = true
            this.uvisa = true
          }
          else {
            this.results.push({eng: "Domestic Violence Protection: Due to your history with domestic abuse or harm at the hands of a family member, you might be eligible for a U-visa. For more information, check Resources.", esp: "Protección contra la violencia doméstica: debido a su historial de abuso o daño doméstico a manos de un miembro de la familia, puede ser elegible para una visa U. Para más información, ver Recursos."})
            this.uvisa = true
          }
        }

        if(this.vicCrime){
          if (!this.reportedCrime){
            this.results.push({eng: "Aid for Victims of Crimes: Since you were the victim of a crime you might be eligible for a U-visa, a T-visa, or support through the Violence Against Women's Act (VAWA). Note that for U-visa eligibility, you must have reported or being will to report the crime and/or cooperate with the investigation or prosecution. For more information, see Resources.", esp: "Ayuda para víctimas de delitos: dado que usted fue víctima de un delito, podría ser elegible para una visa U, una visa T o apoyo a través de la Ley de Violencia contra la Mujer (VAWA). Tenga en cuenta que para la elegibilidad de la visa U, debe haber reportado o estar dispuesto a reportar el delito y / o cooperar con la investigación o el enjuiciamiento. Para más información, ver Recursos."})
            this.vawa = true
            this.uvisa = true
            this.tvisa = true
          }
          else {
            this.results.push({eng: "Aid for Victims of Crimes: Since you were the victim of a crime and reported the crime or helped in its investigation, you might be eligible for a U-visa, a T-visa, or support through the Violence Against Women's Act (VAWA). For more information, see Resources.", esp: "Ayuda para víctimas de delitos: dado que usted fue víctima de un delito y denunció el delito o ayudó en su investigación, puede ser elegible para una visa U, una visa T o apoyo a través de la Ley de Violencia contra la Mujer (VAWA) . Para más información, ver Recursos."})
            this.vawa = true
            this.uvisa = true
            this.tvisa = true
          }
        }

        if(this.recruitWork && (this.trickedToWork || this.exploited)){
          this.results.push({eng: "Aid for Exploited Workers: Since you were recruited to work here in the US and tricked, forced to work, or exploited in some way, you might be eligible for a T-visa. For more information, see Resources.", esp: "Ayuda para trabajadores explotados: dado que fue reclutado para trabajar aquí en los EE.UU. y engañado, obligado a trabajar o explotado de alguna manera, puede ser elegible para una visa T. Para más información, ver Recursos."})
          this.tvisa = true
        }

        if(this.age < 22 && this.neglected) {
          this.results.push({eng: "Juvenile Aid: Considering your age and possible history of abuse, abandonment, or neglect by a parent, you might be eligible for Special Immigrant Juvenile Status. For more information, check Resources.", esp: "Ayuda Juvenil: Teniendo en cuenta su edad y posible historial de abuso, abandono o maltrato por parte de un padre, puede ser elegible para el estatus de Inmigrante Juvenil Especial. Para más información, ver Recursos."})
          this.sijs = true
        }

        if(this.permBar && this.detained){
          this.results.push({eng: "Unlawful Presence: You might be subject to a permanent bar, meaning you cannot re-enter the United States if you leave, because you re-entered the United States after already spending at least a year in the US illegally, leaving the US, and then returning. This only matters if you were caught or detained when exiting or entering. Most applications to become a lawful permanent resident require a consular interview in your country of origin, which can be a major challenge if by going to that meeting you would not be allowed back into the United States. Fortunately, there are some waivers that you might be eligible for, check Resources for more information.", esp: "Presencia ilegal: puede estar sujeto a una prohibición permanente, lo que significa que no puede volver a ingresar a los Estados Unidos si se va, porque volvió a ingresar a los Estados Unidos después de haber pasado al menos un año ilegalmente en los Estados Unidos, saliendo de los Estados Unidos, y luego volviendo. Esto solo importa si fue atrapado o detenido al salir o al entrar. La mayoría de las solicitudes para convertirse en residente legal requieren una entrevista consular en su país de origen, lo que puede ser un gran desafío si al asistir a esa entrevista no se le permitiría regresar a los Estados Unidos. Afortunadamente, hay algunas exenciones para las que puede ser elegible, consulte Recursos para obtener más información."})
          this.waiver = true
        }
        else if(this.tenBar && this.detained){
          this.results.push({eng: "Unlawful Presence: You might be subject to a ten year bar, meaning you cannot re-enter the United States for ten years, because you have spent more than one year in the US unlawfully. This only matters if you were caught or detained when exiting or entering. Most applications to become a lawful permanent resident require a consular interview in your country of origin, which can be a major challenge if by going to that meeting you would not be allowed back into the United States for 10 years. Fortunately, there are some waivers that you might be eligible for. Check Resources for more information.", esp: "Presencia ilegal: puede estar sujeto a una prohibición de diez años, lo que significa que no puede volver a ingresar a los Estados Unidos durante diez años, porque ha pasado más de un año ilegalmente en los Estados Unidos. Esto solo importa si fue atrapado o detenido al salir o al entrar. La mayoría de las solicitudes para convertirse en residente permanente legal requieren una entrevista consular en su país de origen, lo que puede ser un gran desafío si al asistir a esa entrevista no se le permitirá volver a los Estados Unidos durante 10 años. Afortunadamente, hay algunas exenciones para las que puede ser elegible. Consulte Recursos para más información."})
          this.waiver = true
        }
        else if(this.threeBar && this.detained){
          this.results.push({eng: "Unlawful Presence: You might be subject to a three year bar, meaning you cannot re-enter the United States for three years, because you have spent between 6 months and a year in the US unlawfully. This only matters if you were caught or detained when exiting or entering. Most applications to become a lawful permanent resident require a consular interview in your country of origin, which can be a major challenge if by going to that meeting you would not be allowed back into the United States for 3 years. Fortunately, there are some waivers that you might be eligible for: check Resources for more information.", esp: "Presencia ilegal: puede estar sujeto a una prohibición de tres años, lo que significa que no puede volver a ingresar a los Estados Unidos durante tres años, porque ha pasado ilegalmente entre 6 meses y un año en los Estados Unidos. Esto solo importa si fue atrapado o detenido al salir o al entrar. La mayoría de las solicitudes para convertirse en residente permanente legal requieren una entrevista consular en su país de origen, lo que puede ser un gran desafío si al asistir a esa reunión no se le permitirá volver a los Estados Unidos durante 3 años. Afortunadamente, hay algunas exenciones para las que puede ser elegible: consulte Recursos para obtener más información."})
          this.waiver = true
        }

        if(this.results.length === 0) {
          this.results.push({eng: "This current form cannot determine any visas or programs that you qualify for, but note this form is not comprehensive and cannot replace a professional opinion. There are programs not covered by this form because they are too specific, and immigration policy is subject to rapid change.", esp: "Este formulario actual no puede determinar las visas o programas para los que califica, pero tenga en cuenta que este formulario no es exhaustivo y no puede reemplazar una opinión profesional. Hay programas que no están cubiertos por este formulario porque son demasiado específicos, y la política de inmigración está sujeta a cambios rápidos."})
        }
      },
      //TODO: Reset properly
      onReset(evt) {
        evt.preventDefault()
        // Reset our form values
        this.age = ''
        this.household = 4
        this.incomeLevel = ''
        this.insured = null
        this.edLevel = ''
        this.specialSkills = null
        this.unlawfulEntry = null
        this.threeBar = null
        this.tenBar = null
        this.permBar = null
        this.detained = null
        this.parentUS = null
        this.married = null
        this.spouseStatus = ''
        this.children = null
        this.childUSC = null
        this.sibUSC = null
        this.parentLPR = null
        this.arrested = null
        this.wantAsylum = null
        this.warrantAsylum = null
        this.domAbuse = null
        this.abuserUSC = null
        this.threatenerUSC = null
        this.domThreat = null
        this.vicCrime = null
        this.reportedCrime = null
        this.recruitWork = null
        this.trickedToWork = null
        this.exploited = null
        this.neglected = null
        // Trick to reset/clear native browser form validation state
        this.show = false
        this.$nextTick(() => {
          this.show = true
        })
      }
    }
  }
</script>
