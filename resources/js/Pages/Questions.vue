<script setup>
  import Layout from '@/Shared/Layout.vue';
  import NewQuestionModal from '@/Shared/NewQuestionModal.vue'
import { router } from '@inertiajs/vue3';
  import {ref} from 'vue'
  import {usePage} from '@inertiajs/vue3'
  import {computed} from 'vue'
  const page=usePage()
  const success=computed(()=>page.props.flash.success)
  const showNewQuestionModal=ref(false)
  const showViewQuestionModal=ref(false)
  const createdQuestion=ref(null)
  const newAnswers=ref([])
  const answers=ref([])
  const selectedAnswer=ref(null)
  const selectedQuestion=ref(null)
  let answerId=1
  function createQuestion(){
    showNewQuestionModal.value=true
  }

  function destroyModal(){
    showNewQuestionModal.value=false
    showViewQuestionModal.value=false
  }
  function addNewAnswer(){
    const newAnswer={
        id: answerId++,
        answer:'',
        correct_answer:0
    }

    newAnswers.value.push(newAnswer)
  }

  function handleRadioToggle(Id){
     selectedAnswer.value=Id
     newAnswers.value.forEach(answer=>{
        if(answer.id===Id){
            answer.correct_answer=1
        }else{
            answer.correct_answer=0
        }
     })
  }
  function validateAnswers(){
     for(const answer of newAnswers.value){
        if(answer.answer.trim()===''){
            return false
        }
     }
     return true
  }

  function answerCount(){
    if(newAnswers.length<4){
        alert('Four answers required to submit')
    }else if(newAnswers.length===4){
        return true
    }
    return false
  }


  function submitQuestion(){
      if(!createdQuestion.value){
        alert('Question Cannot be empty');
        return false
      }

      if(!validateAnswers() && !answerCount()){
        alert('Fill all inputs before submitting')
        return false
      }

      router.post('/questions',{
        question:createdQuestion.value,
        answers:newAnswers.value
      })

      router.on('success',()=>{
         createdQuestion.value=null,
         newAnswers.value=[]
      })
  }

  const props=defineProps({
     questions: Object,
     errors:Object,
  });

  function viewQuestion(index){
    showViewQuestionModal.value=true
    selectedQuestion.value=props.questions[index]
    answers.value=props.questions[index].answers
      //alert(index);
  }

  //handle radio change and submit edited answers
  const selectedEditAnswer=ref(null)
  function handleRadioChange(Id){
    selectedEditAnswer.value=Id

    answers.value.forEach(answer=>{
      if(answer.id===Id){
        answer.correct_answer=1
      }else{
        answer.correct_answer=0
      }
    })

  }

  //save updated answers to database

  function updateAnswers(){
    router.put('/answers',
     answers.value
    )
  }

  //edit question
  const editquestionModal=ref(false)
  const questionForEdit=ref(null)
  function editQuestion(index){
    questionForEdit.value=props.questions[index]
    //alert(index)
  }

  function updateQuestion(){
     router.put('/questions',
     questionForEdit.value
     )
  }

  //delete question
 function deleteQuestion(id){
   router.on('before',()=>{
    return confirm('You are about to delete a record, are you sure')
   })

   router.delete('/questions/'+id)
 }
</script>
<template>
  <Layout>
    <button @click="createQuestion" class="btn btn-success">Create</button>
    <table class="table table-striped">
  <thead>
    <tr>
      <th scope="col">#</th>
      <th scope="col">Question</th>
      <th scope="col">Action</th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="(question,index) in questions">
      <th scope="row">{{index+1}}</th>
      <td>{{question.question }}</td>
      <td>
        <button @click="viewQuestion(index)" class="btn btn-primary">View</button>
        <button @click="editquestionModal=true,editQuestion(index)" class="btn btn-success">Edit</button>
        <button @click="deleteQuestion(question.id)" class="btn btn-danger">Delete</button>
      </td>

    </tr>
    </tbody>
    </table>


    <Teleport to="body">
       <NewQuestionModal :show="showNewQuestionModal" @close="destroyModal">
        <template #header>
            <h5>Add New Question</h5>

        </template>
        <template #success>
          <div v-if="success" class="alert alert-success">{{success}}</div>
        </template>
        <template #body>
            <form>
            <div class="mb-3">
                <label for="exampleInputEmail1" class="form-label"> Question</label>
                <input type="text" v-model="createdQuestion" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp">
            </div>
            <table class="table">
  <thead>
    <tr>
      <th scope="col">#</th>
      <th scope="col">Answers</th>
      <th scope="col">Correct ?</th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="(answer,index) in newAnswers">
      <th scope="row">{{answer.id}}</th>
      <td>
        <input type="text" v-model="answer.answer" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp">
      </td>
      <td>
        <input :checked="answer.correct_answer ===1" class="form-check-input" :value="answer.id" @change="handleRadioToggle(answer.id)" type="radio">
      </td>
    </tr>
   </tbody>
   </table>
            </form>
          </template>
          <template #footer>
            <span @click="addNewAnswer" v-if="newAnswers.length<4"><h3>+</h3></span>
            <button @click="destroyModal" class="btn btn-danger">Close</button>
            <button v-if="newAnswers.length>3" @click="submitQuestion" class="btn btn-success">Submit</button>
        </template>
       </NewQuestionModal>

       <NewQuestionModal :show="showViewQuestionModal" @close="destroyModal">
        <template #header>
         <h5>View Question/Answers</h5>
        </template>
        <template #success>
          <div v-if="success" class="alert alert-success">{{success}}</div>
        </template>
        <template #body>
           <p><strong>Q.{{selectedQuestion.question}}</strong></p>

          <table class="table">
          <thead>
            <tr>
              <th scope="col">#</th>
              <th scope="col">Answers</th>
              <th scope="col">Correct ?</th>
            </tr>
          </thead>
            <tbody>
            <tr v-for="(answer,index) in answers">
              <th scope="row">{{index+1}}</th>
              <td>
                <input type="text" v-model="answer.answer" class="form-control" id="exampleInputEmail1" aria-describedby="emailHelp">
              </td>
              <td>
                <input :checked="answer.correct_answer ===1" class="form-check-input" :value="answer.id" @change="handleRadioChange(answer.id)" type="radio">
              </td>
            </tr>
          </tbody>
          </table>
        </template>
        <template #footer>
           <button @click="destroyModal" class="btn btn-danger">Close</button>
           <button @click="updateAnswers"  class="btn btn-success">Update</button>
        </template>
      </NewQuestionModal>

      <NewQuestionModal :show="editquestionModal" @click="aditQuestionModal=false">
        <template #header>
          <h5>Edit Question</h5>
        </template>;
        <template #success>
          <div v-if="success" class="alert alert-success">{{success}}</div>
        </template>
        <template #body>
          <div class="mb-3">
            <label for="exampleFormControlInput1" class="form-label">Question</label>
            <input v-model="questionForEdit.question" type="email" class="form-control" id="exampleFormControlInput1" placeholder="name@example.com">
          </div>
        </template>
        <template #footer>
          <button @click="editquestionModal=false" class="btn btn-danger">Close</button>
           <button @click="updateQuestion"  class="btn btn-success">Update</button>
        </template>
      </NewQuestionModal>
    </Teleport>
  </Layout>
</template>

Imports :

Layout : Il s'agit d'un composant de mise en page partagé utilisé pour envelopper le contenu de cette page.
NewQuestionModal : Un composant pour afficher une fenêtre modale lors de la création ou de l'édition de questions.
router : Il provient de la bibliothèque Inertia.js et est utilisé pour gérer les routes et les interactions avec le backend.
ref : Il provient de Vue 3 et est utilisé pour créer des variables réactives.
usePage : Un hook fourni par Inertia.js pour accéder aux données de la page courante.
computed : Il provient de Vue 3 et est utilisé pour créer des propriétés calculées.
Variables réactives et fonctions :

page : Stocke les données de la page actuelle obtenues à partir du hook usePage.
success : Une propriété calculée qui extrait la propriété flash.success des données de la page.
showNewQuestionModal, showViewQuestionModal : Des variables réactives pour contrôler l'affichage des modales.
createdQuestion : Stocke la question en cours de création.
newAnswers, answers : Tableaux réactifs pour stocker les réponses en cours de création et les réponses d'une question existante.
selectedAnswer, selectedEditAnswer : Stockent l'ID de la réponse sélectionnée comme réponse correcte.
selectedQuestion : Stocke la question sélectionnée pour afficher ses réponses.
answerId : Compteur pour attribuer des IDs uniques aux nouvelles réponses.
Fonctions :

createQuestion, destroyModal : Gèrent l'affichage et la fermeture de la modal de création de question.
addNewAnswer : Ajoute une nouvelle réponse au tableau de réponses en cours de création.
handleRadioToggle : Gère la sélection d'une réponse comme réponse correcte lors de la création de la question.
validateAnswers : Vérifie si toutes les réponses en cours de création ont du contenu.
answerCount : Vérifie si au moins quatre réponses sont présentes pour soumettre la question.
submitQuestion : Soumet la question et ses réponses au backend via une requête POST.
viewQuestion : Affiche la modal de visualisation de la question et initialise les données pour l'affichage.
handleRadioChange : Gère la sélection d'une réponse comme réponse correcte lors de la modification de la question.
updateAnswers : Soumet les réponses mises à jour au backend via une requête PUT.
editQuestion, updateQuestion : Gèrent la modification d'une question existante et sa soumission au backend.
deleteQuestion : Demande une confirmation avant de supprimer une question.
Template :

Le composant utilise des balises HTML standard, des directives Vue (v-for, v-model, @click, etc.)
et des classes Bootstrap pour styliser l'interface utilisateur.
Il affiche une liste de questions dans un tableau avec des boutons pour afficher, éditer et supprimer chaque question.
Des modales sont téléportées dans le DOM pour afficher les formulaires de création, de visualisation et d'édition de questions.
En résumé, le composant "question.vue" gère l'interface utilisateur pour la création, la visualisation, l'édition et
la suppression de questions et de réponses dans le contexte d'une application Laravel en utilisant Vue.js et Inertia.js.
Il facilite l'interaction entre le frontend et le backend en utilisant des routes et des modales
pour fournir une expérience utilisateur fluide.
