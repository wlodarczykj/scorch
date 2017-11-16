<template>
    <div>
        <div class="dm-quiver-creator border border-dark">
            <h4> Quiver </h4>
            <div class="quiver-form">
                <div class="form-group">
                    <label for="name">Name : </label>
                    <input type="text" class="form-control" id="name" v-model="name" placeholder="Name" autocomplete="off" required="true"/>
                </div>
                <div class="form-group">
                    <label for="description">Description : </label>
                    <textarea rows="4" class="form-control" id="description" v-model="description" placeholder="Description" autocomplete="off" required="true"/>
                </div>
                <div class="form-group numeric-entry">
                    <label for="item-type">Item Type : </label>
                    <input type="text" class="form-control" id="item-type" v-model="itemType" placeholder="Item Type" autocomplete="off" required="true"/>
                </div>
                <div class="d-flex">
                    <div class="form-group numeric-entry">
                        <label for="weight">Weight : </label>
                        <div class="input-group">
                            <input type="text" class="form-control" id="weight" v-model="weight" placeholder="Weight" autocomplete="off" required="true"/>
                            <span class="input-group-addon">lbs</span>
                        </div>
                    </div>
                    <div class="form-group numeric-entry">
                        <label for="cost">Cost : </label>
                        <div class="input-group">
                            <input type="number" class="form-control" id="cost" v-model="cost" placeholder="Cost in gp" autocomplete="off" required="true"/>
                            <span class="input-group-addon">gp</span>
                        </div>
                    </div>
                </div>
                <div class="max-ammo-entry">
                    <label class="max-ammo-label">Max Ammo</label>
                    <div class="max-ammo-holder">
                        <ul class="list-group">
                            <div class="max-ammo-list-items" v-for="(maxAmmo, index) in maxAmmoList" :key="index">
                                <li class="list-group-item">{{maxAmmo.Type + " : " + maxAmmo.MaxAmount}}</li>
                            </div>
                        </ul>
                    </div>
                    <div class="input-group">
                        <button class="btn btn-primary add-remove-btn" type="button" v-on:click="addAmmo()"><b>+</b></button>
                        <button class="btn btn-danger add-remove-btn" type="button" v-on:click="removeAmmo()"><b>-</b></button>
                        <input type="number" class="form-control ammo-amt" id="ammo-input-amt" v-model="newAmmoMaxAmt" placeholder="Count" autocomplete="off"/>
                        <input type="text" class="form-control" id="ammo-input-type" v-model="newAmmoType" placeholder="Type" autocomplete="off"/>
                    </div>
                </div>
                <div class="properties">
                    <label class="property-label">Properties</label>
                    <div class="property-holder">
                        <div class="property-list-items" v-for="(prop, index) in properties" :key="index">
                            <span class="badge-small badge-pill badge-secondary">{{prop}}</span>
                        </div>
                    </div>
                    <div class="input-group">
                        <button class="btn btn-primary add-remove-btn" type="button" v-on:click="addProp()"><b>+</b></button>
                        <button class="btn btn-danger add-remove-btn" type="button" v-on:click="removeProp()"><b>-</b></button>
                        <input type="text" class="form-control" id="property-input" v-model="newProp" placeholder="Properties" autocomplete="off"/>
                    </div>
                </div>
                <button class="btn btn-primary" @click="create()">Submit</button>
                <button class="btn btn-primary" @click="update()" v-if="itemId">Update</button>
                <button class="btn btn-primary" @click="deleteItem()" v-if="itemId">Delete</button>
                <button class="btn btn-danger clear-button" type="button" @click="clearFields()">Clear</button>
            </div>
        </div>

    </div>
</template>

<script>
    import { ItemService } from 'services'

export default {
    name: 'dm-quiver-creator',
    props: ['quiver'],
    data(){
        return {
            description : this.quiver.Description || '',
            newAmmoType: '',
            itemType : this.quiver.ItemType || '',
            newProp: '',
            name : this.quiver.Name || '',
            newAmmoMaxAmt : 0,
            weight : this.quiver.Weight || 0,
            cost : this.quiver.Cost || 0,
            properties: this.quiver.Properties || [],
            maxAmmoList: [],
            itemId: this.quiver.ItemId || ''
        }
    },
    watch: {
      quiver: function () {
        this.clearFields();
        this.description = this.quiver.Description;
        this.itemType = this.quiver.ItemType;
        this.name = this.quiver.Name;
        this.newAmmoMaxAmt = 0;
        this.weight = this.quiver.Weight;
        this.cost = this.quiver.Cost;
        this.properties = this.quiver.Properties;
        this.maxAmmoList = [];
        this.newAmmoType = '';
        this.newProp = '';
        this.itemId = this.quiver.ItemId;
      }
    },
    methods: {
        addProp() {
            let isPresent = this.properties.includes(this.newProp);
            if(this.newProp && !isPresent){
                this.properties.push(this.newProp);
                this.newProp = '';
            }
        },
        removeProp(){
            let index = this.properties.indexOf(this.newProp);
            if(index >= 0 && this.newProp){
                this.properties.splice(index, 1);
                this.newProp = '';
            } else {
                this.properties.pop();
            }
        },
        addAmmo(){
            let isPresent = this.maxAmmoList.find(ammo => ammo.Type == this.newAmmoType);
            if(this.newAmmoMaxAmt > 0)
            {
                if(isPresent){
                    this.removeAmmo();
                }
                let newAmmo = {};
                newAmmo.Type = this.newAmmoType;
                newAmmo.MaxAmount = this.newAmmoMaxAmt;
                this.maxAmmoList.push(newAmmo);
                this.newAmmoType = '';
                this.newAmmoMaxAmt = 0;
            }
        },
        removeAmmo(){
            let index = this.maxAmmoList.findIndex(ammo => ammo.Type == this.newAmmoType);
            if(index >= 0){
                this.maxAmmoList.splice(index, 1);
                this.newAmmoType = '';
            } else {
                this.maxAmmoList.pop();
            }
        },
        buildPayload() {
            let payload = {};
            let body = {};
            body.ItemClass = 'Quiver';

            body.Name = this.name;
            body.Cost = this.cost;
            body.Slot = "Quiver";
            body.Weight = this.weight;
            body.Properties = this.properties;
            body.Description = this.description;
            body.ItemType = this.itemType;

            let maxAmmoPayload = {};
            this.maxAmmoList.forEach(ammo => {
                maxAmmoPayload[ammo.Type] = ammo.MaxAmount;
            });
            body.Projectiles = maxAmmoPayload;

            payload.body = body;
            return payload;
        },
        async create(){
            let payload = buildPayload();
            await this.$store.dispatch('addItem', payload);
            if(this.$store.getters.error){
                console.log("Encountered an error during item creation : " + this.error);

                this.$notify.failure(`Error encountered while creating ${this.name}`);
            }
            else{
                this.$notify.success(`Successfully created ${this.name}`);
                this.clearFields();
            }

        },
        async update() {
            let payload = this.buildPayload();
            payload.body.ItemId = this.itemId;
            await this.$store.dispatch('updateInventory', payload);

            if(this.$store.getters.error){
                console.log("Encountered an error during item update : " + this.error);

                this.$notify.failure(`Error encountered while updating ${this.name}`);
            }
            else{
                this.$notify.success(`Successfully updated ${this.name}`);
                this.clearFields();
            }
        },
        async deleteItem() {
            let response = {};
            try{
                response = await ItemService.deleteItem(this.itemId);
                this.$notify.success(`Successfully deleted ${this.name}`);
            }
            catch(errorResponse){
                console.log(`Failed to delete ${this.name}. Error: ${errorResponse.bodyText}`);
                this.$notify.failure(`Failed to delete ${this.name}`);
            }
        },
        clearFields(){
            this.description = '';
            this.newAmmoType = '';
            this.itemType = '';
            this.newProp = '';
            this.name = '';
            this.newAmmoMaxAmt = 0;
            this.weight = 0;
            this.cost = 0;
            this.properties = [];
        }
    }
}
</script>

<style lang="scss" scoped>
    .dm-quiver-creator {
        margin: 1%;
        margin-top: 2%;
        padding: 1%;
        border-radius: 10px;
    }
    .quiver-form{
        margin : 3%;
    }
    .properties{
        margin-bottom: 5%;
    }
    .max-ammo-entry{
        margin-bottom: 2%;
    }
    .property-list-items {
        float:left;
        font-size:medium;
    }
    .property-holder {
        padding:1%;
    }
    .max-ammo-holder {
        margin-bottom: 5%;
    }
    .numeric-entry {
        padding-right:2%;
        flex: 1;
    }
    .property-label {
        margin-bottom:1%;
    }
    .success-notification{
        display: none;
        position: absolute;
        width: 88%;
        margin-top: -4%;
    }
    .failure-notification{
        display: none;
        position: absolute;
        width: 88%;
        margin-top: -4%;
    }
    .clear-button{
        float: right;
    }
    .ammo-amt{
        width: 0%;
    }
    .add-remove-btn{
        width: 10%;
    }
</style>