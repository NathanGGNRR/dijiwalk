<template>
    <q-page class="q-px-xl">
        <q-header elevated>
            <q-toolbar class="bg-toolbar">


                <q-btn flat round color="white" class="q-ml-md cursor-pointer" icon="fas fa-arrow-left" v-go-back=" '/' " />
                <div class="row full-width justify-center">
                    <img src="https://storage.cloud.google.com/dijiwalk-test/logo-text.png" style="max-height:50px;" class="q-my-sm" />
                </div>

            </q-toolbar>
        </q-header>
        <div class="row full-width justify-center q-pr-xl q-my-md q-col-gutter-xl">
            <div class="col-xs-12 col-md-4 col-grow">
                <q-card @click="openModalToAdd()" class="my-card full-height cursor-pointer flex column justify-center items-center bg-negative first-card q-py-md">
                    <q-icon name="fas fa-plus text-white" style="font-size: 3vw;" />
                </q-card>
            </div>
            <div v-for="participant in participants" v-bind:key="participant.id" class="col-xs-12 col-md-4 col-grow">
                <q-card class="my-card">

                    <div @click="openModalToEdit(participant)" class="game-card">
                        <q-img :src="participant.picture" />

                        <q-card-section>
                            <q-btn v-on:click.stop="openModalToDelete(participant)"
                                   fab
                                   color="negative"
                                   icon="fas fa-trash"
                                   class="absolute"
                                   style="top: 0; right: 12px; transform: translateY(-50%); z-index: 999;" />

                            <div class="row no-wrap">
                                <div class="col text-left text-bold text-h6 ellipsis">
                                    {{ participant.firstName }} {{ participant.lastName }} / {{ participant.Login }}
                                </div>
                            </div>
                            <div class="row items-center no-wrap text-grey">
                                <q-icon name="fas fa-users" />
                                <p class="q-ma-none q-ml-sm">{{ participant.email }}</p>
                            </div>
                        </q-card-section>
                    </div>

                    <q-separator />

                    <q-card-actions>
                        <q-btn flat @click="openModalToGetInformations(participant)" class="dijiwalk-secondary" round icon="fas fa-info-circle" />
                        <q-btn flat @click="openModalToGetInformations(participant)" class="dijiwalk-secondary" color="primary text-bold">
                            Informations
                        </q-btn>
                    </q-card-actions>
                </q-card>
            </div>
        </div>

        <q-dialog v-model="confirm">
            <q-card>
                <q-card-section class="row items-center">
                    <div class="col-2">
                        <q-avatar icon="fas fa-exclamation-triangle" color="negative" text-color="white" size="70px" />
                    </div>
                    <div class="row col-10">
                        <span>?tes-vous s?r de vouloir supprimer le participant {{ messageDeleteParticipant }}</span>
                        <span class="text-caption text-negative q-mt-sm" color="negative" style="font-size: 10px;">{{ messageBonus }}</span>
                    </div>
                </q-card-section>

                <q-card-actions align="right">
                    <q-btn flat label="Annuler" color="primary" v-close-popup />
                    <q-btn flat label="Supprimer" color="negative" @click="deletedParticipant" v-close-popup />
                </q-card-actions>
            </q-card>
        </q-dialog>
        <q-dialog v-model="manageParticipant">
            <q-card>
                <q-card-section class="row items-center">
                    <q-input v-if="isEditing" v-model="idParticipant" type="hidden" />
                    <q-input ref="firstname" class="col-6 q-pr-sm  q-my-xs" label="Pr?nom *" color="primary" option-value="id" option-label="name" v-model="prenomParticipant" name="prenomParticipant" id="prenomParticipant" lazy-rules :rules="[ val => val && val.length > 0 || 'Veuillez renseigner un pr?nom.']">
                        <template v-slot:prepend>
                            <q-icon name="fas fa-address-card" />
                        </template>
                    </q-input>
                    <q-input ref="lastname" class="col-6 q-pl-sm q-my-xs" label="Nom *" color="primary" option-value="id" option-label="name" v-model="nomParticipant" name="nomParticipant" id="nomParticipant" lazy-rules :rules="[ val => val && val.length > 0 || 'Veuillez renseigner un nom.']">
                        <template v-slot:prepend>
                            <q-icon name="fas fa-address-card" />
                        </template>
                    </q-input>
                    <q-input ref="login" class="col-12 q-my-xs" label="Login *" color="primary" option-value="id" option-label="name" v-model="loginParticipant" name="loginParticipant" id="loginParticipant" lazy-rules :rules="[ val => val && val.length > 0 || 'Veuillez renseigner un pseudo.']" :error-message="errormessagelogin" :error="errorlogin">
                        <template v-slot:prepend>
                            <q-icon name="fas fa-user" />
                        </template>
                    </q-input>
                    <q-input class="col-12 q-my-xs" ref="password" label="Mot de passe *" v-model="passwordParticipant" :type="isPwd ? 'password' : 'text'" name="passwordParticipant" id="passwordParticipant" :error-message="errormessagepassword" :error="errorpassword" lazy-rules :rules="[ val => val && val.length > 0 || 'Veuillez renseigner un mot de passe si nouveau participant.']">
                        <template v-slot:prepend>
                            <q-icon name="fas fa-lock" />
                        </template>
                        <template v-slot:append>
                            <q-icon :name="isPwd ? 'fas fa-eye-slash' : 'fas fa-eye'"
                                    class="cursor-pointer"
                                    @click="isPwd = !isPwd" />
                        </template>
                    </q-input>
                    <q-input ref="confirmPassword" class="col-12 q-my-xs" label="Mot de passe de confirmation *" v-model="passwordConfirmParticipant" :type="isPwd ? 'password' : 'text'" name="passwordConfirmParticipant" id="passwordConfirmParticipant" :error-message="errormessagepassword" :error="errorpassword" lazy-rules :rules="[ val => val && val.length > 0 || 'Veuillez confirmer le mot de passe si nouveau participant.']">
                        <template v-slot:prepend>
                            <q-icon name="fas fa-lock" />
                        </template>
                        <template v-slot:append>
                            <q-icon :name="isPwd ? 'fas fa-eye-slash' : 'fas fa-eye'"
                                    class="cursor-pointer"
                                    @click="isPwd = !isPwd" />
                        </template>
                    </q-input>
                    <q-input class="col-12 q-my-xs" ref="email" label="Email *" color="primary" option-value="id" option-label="name" v-model="emailParticipant" name="emailParticipant" type="email" id="emailParticipant" lazy-rules :rules="[ val => val && val.length > 0 || 'Veuillez renseigner un email.']" :error-message="errormessageemail" :error="erroremail">
                        <template v-slot:prepend>
                            <q-icon name="fas fa-at" />
                        </template>
                    </q-input>
                    <q-file class="col-12 q-my-xs" ref="picture" v-model="pictureParticipant" label="Image de profil *" accept=".jpg, image/*" lazy-rules :rules="[val => !!val || 'Image obligatoire si nouveau participant !']" clearable @input="takePicture" @clear="clearPicture">
                        <template v-slot:prepend>
                            <q-avatar id="avatarSelected" v-show="!noPicture">

                            </q-avatar>
                            <q-icon v-show="noPicture" name="fas fa-image" />
                        </template>
                    </q-file>
                </q-card-section>
                <q-card-actions align="right">
                    <q-btn flat label="Annuler" color="primary" v-close-popup />
                    <q-btn flat v-if="isEditing" label="Modifier" @click="updateParticipant" color="secondary" />
                    <q-btn v-if="isAdding" label="Ajouter" @click="addedParticipant" color="secondary" />
                </q-card-actions>
            </q-card>
        </q-dialog>

        <q-dialog v-if="participantSelected !== null" v-model="informations">
            <q-card class="modal-informations">
                <q-card-section class="items-center">
                    <q-img :src="participantSelected.picture" class="img-radius row items-center justify-center" />
                    <h5 class="q-my-sm">{{ participantSelected.firstName }} {{ participantSelected.lastName }}</h5>
                    <p>{{ participantSelected.Login }} / {{ participantSelected.email }}</p>
                </q-card-section>

                <q-card-actions align="right">
                    <q-btn flat label="Annuler" color="primary" v-close-popup />
                </q-card-actions>
            </q-card>
        </q-dialog>

    </q-page>
</template>

<script>

    import PlayerDataService from '../services/PlayerDataService';

    export default {
        name: 'participant',
        data() {
            return {
                modelEquipe: null,
                modelJeu: null,
                equipesOptions: null,
                jeuxOptions: null,

                participants: null,
                confirm: false,

                isEditing: false,
                isAdding: false,
                informations: false,

                manageParticipant: false,
                messageDeleteParticipant: null,
                messageBonus: "Si vous supprimer un participant, cela supprimera ses liens avec les ?quipes !",
                deleteParticipant: null,

                participantSelected: null,
                prenomParticipant: null,
                nomParticipant: null,
                loginParticipant: null,
                passwordParticipant: null,
                passwordConfirmParticipant: null,
                emailParticipant: null,
                pictureParticipant: null,
                pictureUrl: null,

                errormessagepassword: null,
                errorpassword: false,
                errormessageemail: null,
                erroremail: false,
                errormessagelogin: null,
                errorlogin: false,

                isPwd: true,
                idParticipant: null,
                encryptedPassword: null,
                noPicture: true
            }
        },
        created() {

            this.getAllParticipants();

        },

        methods: {
            fileConvert() {
                return new Promise((resolve, reject) => {
                    if (this.pictureParticipant != null) {
                        const reader = new FileReader();
                        reader.readAsDataURL(this.pictureParticipant);
                        reader.onload = () => resolve(reader.result);
                        reader.onerror = error => reject(error);
                    } else {
                        resolve(this.pictureParticipant);
                    }
                })
            },
            clearPicture() {
                document.getElementById('avatarSelected').innerHTML = "";
                this.noPicture = true;
            },
            takePicture() {
                if (this.pictureParticipant != null && "name" in this.pictureParticipant) {
                    this.noPicture = false;
                    let reader = new FileReader();
                    reader.onload = function (e) {
                        if (e.target.result.indexOf("image") != -1) {
                            document.getElementById('avatarSelected').innerHTML = ['<img style="width:35px; height: 35px;" src="', e.target.result, '" />'].join('')
                        }
                    };
                    reader.readAsDataURL(this.pictureParticipant);
                }
            },
            openModalToGetInformations(participant) {
                this.isAdding = false;
                this.isEditing = false;
                this.participantSelected = participant;
                this.informations = true;
            },

            openModalToAdd() {
                this.isEditing = false;
                this.isAdding = true;
                this.resetInput();
                this.manageParticipant = true;
            },

            openModalToEdit(participant) {
                this.isEditing = true;
                this.isAdding = false;
                this.resetInput();
                this.fillForm(participant);
                this.manageParticipant = true
            },
            resetInput() {
                this.idParticipant = null
                this.prenomParticipant = null
                this.nomParticipant = null
                this.loginParticipant = null
                this.passwordParticipant = null
                this.passwordConfirmParticipant = null
                this.emailParticipant = null
                this.pictureParticipant = null
            },
            fillForm(participant) {
                this.participantSelected = participant
                this.idParticipant = participant.id
                this.prenomParticipant = participant.firstName
                this.nomParticipant = participant.lastName
                this.loginParticipant = participant.Login
                this.passwordParticipant = null
                this.passwordConfirmParticipant = null
                this.emailParticipant = participant.email
                this.pictureUrl = participant.picture
                this.encryptedPassword = participant.Password
            },
            getAllParticipants() {
                this.$q.loading.show()
                if (this.participants === null) {
                    PlayerDataService.getAll().then(response => {
                        this.participants = response.data;
                        this.$q.loading.hide()
                    }).catch();
                }
            },
            onResetValidation() {
                this.errorpassword = false
                this.erroremail = false
                this.errorlogin = false
                this.errormessagepassword = null
                this.errormessageemail = null
                this.errormessagelogin = null
            },
            openModalToDelete(participant) {
                this.confirm = true;
                this.messageDeleteParticipant = participant.firstName + " " + participant.lastName;
                this.deleteParticipant = participant.id;
            },
            addedParticipant() {
                if (this.$refs.firstname.validate() && this.$refs.lastname.validate() && this.$refs.login.validate() && this.$refs.password.validate() && this.$refs.confirmPassword.validate() && this.$refs.email.validate() && this.$refs.picture.validate()) {
                    if (this.passwordParticipant === this.passwordConfirmParticipant) {
                        if (new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\/$%\/^&\/*])(?=.{8,})').test(this.passwordParticipant)) {
                            if (/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(this.emailParticipant)) {
                                this.$q.loading.show()
                                this.fileConvert().then(response => {
                                    PlayerDataService.create({
                                        FirstName: this.prenomParticipant,
                                        LastName: this.nomParticipant,
                                        Login: this.loginParticipant,
                                        Password: this.passwordParticipant,
                                        Email: this.emailParticipant,
                                        ImageBase64: response,
                                        ImageChanged: true,
                                        PasswordChanged: true
                                    }).then(response => {
                                        this.$q.loading.hide()
                                        if (response.data.status == 1) {
                                            this.manageParticipant = false;
                                            this.onResetValidation();
                                            this.participants.push(response.data.response);
                                            this.$q.notify({
                                                icon: 'fas fa-check-square',
                                                color: 'secondary',
                                                message: response.data.message,
                                                position: 'top'
                                            })
                                        } else {
                                            var message = response.data.message;
                                            this.errorlogin = true;
                                            this.errormessagelogin = message;
                                            this.erroremail = true;
                                            this.errormessageemail = message;
                                            this.$q.notify({
                                                icon: 'fas fa-exclamation-triangle',
                                                color: 'negative',
                                                message: message,
                                                position: 'top'
                                            })
                                            setTimeout(this.onResetValidation, 3000);
                                        }
                                    }).catch();
                                })

                            } else {
                                this.erroremail = true;
                                this.errormessageemail = "Veuillez entrer un email valide.";
                                setTimeout(this.onResetValidation, 3000);
                            }
                        } else {
                            this.errorpassword = true;
                            this.errormessagepassword = "Il faut au moins 8 caract?res, une majuscule, une minuscule, un nombre et un caract?re sp?ciale.";
                            setTimeout(this.onResetValidation, 3000);
                        }
                    } else {
                        this.errorpassword = true;
                        this.errormessagepassword = "Les mots de passes ne correspondent pas !";
                        setTimeout(this.onResetValidation, 3000);
                    }
                }
            },
            updateParticipant() {
                if (this.$refs.firstname.validate() && this.$refs.lastname.validate() && this.$refs.login.validate() && this.$refs.email.validate()) {
                    if (this.passwordParticipant === this.passwordConfirmParticipant) {
                        if (new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\/$%\/^&\/*])(?=.{8,})').test(this.passwordParticipant) || this.passwordParticipant == null) {
                            if (/^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(this.emailParticipant)) {
                                this.$q.loading.show()
                                this.fileConvert().then(response => {
                                    PlayerDataService.update(this.idParticipant, {
                                        FirstName: this.prenomParticipant,
                                        LastName: this.nomParticipant,
                                        Login: this.loginParticipant,
                                        Password: this.encryptedPassword,
                                        Email: this.emailParticipant,
                                        ImageBase64: this.pictureParticipant != null ? response : this.pictureUrl,
                                        ImageChanged: this.pictureParticipant != null ? true : false,
                                        Picture: this.pictureUrl,
                                        PasswordChanged: this.passwordParticipant != null ? true : false
                                    }).then(response => {
                                       this.$q.loading.hide()
                                        if (response.data.status == 1) {
                                            this.manageParticipant = false;
                                            this.onResetValidation();
                                            this.participants[this.participants.map(e => e.id).indexOf(this.participantSelected.id)] = response.data.response

                                            this.$q.notify({
                                                icon: 'fas fa-check-square',
                                                color: 'secondary',
                                                message: response.data.message,
                                                position: 'top'
                                            })
                                        } else {
                                            var message = response.data.message;
                                            this.errorlogin = true;
                                            this.errormessagelogin = message;
                                            this.erroremail = true;
                                            this.errormessageemail = message;
                                            this.$q.notify({
                                                icon: 'fas fa-exclamation-triangle',
                                                color: 'negative',
                                                message: message,
                                                position: 'top'
                                            })
                                            setTimeout(this.onResetValidation, 3000);
                                        }
                                    }).catch();
                                })

                            } else {
                                this.erroremail = true;
                                this.errormessageemail = "Veuillez entrer un email valide.";
                                setTimeout(this.onResetValidation, 3000);
                            }
                        } else {
                            this.errorpassword = true;
                            this.errormessagepassword = "Il faut au moins 8 caract?res, une majuscule, une minuscule, un nombre et un caract?re sp?ciale.";
                            setTimeout(this.onResetValidation, 3000);
                        }
                    } else {
                        this.errorpassword = true;
                        this.errormessagepassword = "Les mots de passes ne correspondent pas !";
                        setTimeout(this.onResetValidation, 3000);
                    }
                }
            },
            onReset() {
                this.prenomParticipant = null
                this.nomParticipant = null
                this.loginParticipant = null
                this.passwordParticipant = null
                this.passwordConfirmParticipant = null
                this.emailParticipant = null
            },
            deletedParticipant() {
                var self = this;
                var id = self.deleteParticipant;
                this.$q.loading.show()
                PlayerDataService.delete(id).then(response => {
                    this.$q.loading.hide()
                    if (response.data.status == 1) {
                        self.$q.notify({
                            icon: 'fas fa-check-square',
                            color: 'secondary',
                            message: response.data.message,
                            position: 'top'
                        })
                        self.participants = self.participants.filter(function (obj) {
                            return obj.id !== self.deleteParticipant;

                        });
                    } else {
                        self.$q.notify({
                            message: response.data.message,
                            color: 'negative',
                            icon: 'fas fa-exclamation-triangle',
                            position: 'top'
                        })
                    }

                }).catch();
            },
            filterEquipe(val, update) {
                update(() => {
                    if (val === '') {
                        this.equipesOptions = this.equipes
                    }
                    else {
                        const needle = val.toLowerCase()
                        this.equipesOptions = this.equipes.filter(
                            v => v.toLowerCase().indexOf(needle) > -1
                        )
                    }
                })
            },
            filterJeux(val, update) {
                update(() => {
                    if (val === '') {
                        this.jeuxOptions = this.jeux
                    }
                    else {
                        const needle = val.toLowerCase()
                        this.jeuxOptions = this.jeux.filter(
                            v => v.toLowerCase().indexOf(needle) > -1
                        )
                    }
                })
            }
        }
    }
</script>

<style scoped>
    .first-card:hover {
        background-color: #cc0016 !important;
    }
</style>