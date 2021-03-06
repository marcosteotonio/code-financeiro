<template>
    <div class="container">
        <div class="row">
            <page-title>
                <h5>Administração de Categorias</h5>
            </page-title>
            <div class="card-panel z-depth-5">
                <category-tree :categories="categories"></category-tree>
            </div>

            <category-save :modal-options="modalOptionSave"
                           :category.sync="categorySave"
                           :cp-options="cpOptions"
                           @save-category="saveCategory()">
                <span slot="title">{{title}}</span>
                <div slot="footer">
                    <button type="submit" class="btn btn-flat waves-effect green lighten-2 modal-close modal-action">OK</button>
                    <button type="button" class="btn btn-flat waves-effect waves-red modal-close modal-action">Cancelar</button>
                </div>
            </category-save>
            <div class="fixed-action-btn">
                <button class="btn-floating btn-large" @click="modalNew(null)">
                    <i class="large material-icons">add</i>
                </button>
            </div>
        </div>
    </div>
</template>


<script type="text/javascript">
    import PageTitleComponent from '../../../../_default/components/PageTitle.vue';
    import CategoryTreeComponent from './CategoryTree.vue'
    import CategorySaveComponent from './CategorySave.vue';
    import {Category} from '../../services/resources';
    import {CategoryFormat, CategoryService} from '../../services/category-nsm';

    export default{
        components:{
            'page-title': PageTitleComponent,
            'category-tree': CategoryTreeComponent,
            'category-save': CategorySaveComponent,
        },
        data(){
            return {
                categories:[],
                categoriesFormatted:[],
                categorySave:{
                    id: 0,
                    name: '',
                    parent_id: 0,
                },
                category: null,
                parent: null,
                title: '',
                modalOptionSave: {
                    id: 'modal-category-save'
                },


            }
        },
        computed:{
            //opções para o campo select2 de categoria pai
          cpOptions(){
              return {
                  data: this.categoriesFormatted,
                  templateResult(category){
                      // margem das categorias conforme o seu nivel de parentesco
                      let margin = '&nbsp'.repeat(category.level * 6) // se for do nivel 1 terá 6 espaçes em branco se for segundo filho terá 12 espaçoes em branco (6x1 e 6x2)
                      let text = category.hasChildren ? `<strong>${category.text}</strong>` : category.text; // se a categoria tiver filho escreve em negrito, se a categoria não tiver filhos escreve sem ser negrtito
                      return `${margin}${text}`;
                  },
                  escapeMarkup(m){
                      return m;
                  }
              }
            }
        },
        created(){
            this.getCategories();
        },
        methods:{
            getCategories(){
                Category.query().then(response => {
                    this.categories = response.data.data;
                    this.formatCategories();
                })
            },
            saveCategory(){
                CategoryService.save(this.categorySave, this.parent, this.categories, this.category).then(response =>{
                    if(this.categorySave.id === 0){
                        Materialize.toast('categoria adicionada com sucesso', 4000);
                    }else{
                        Materialize.toast('categoria alterada com sucesso', 4000);
                    }
                    this.resetScope();
                });
            },
            modalNew(category){
                this.title = 'Nova categoria'
                this.categorySave = {
                    id: 0,
                    name: '',
                    parent_id: category === null ? null : category.id,
                };
                this.parent = category;
                $(`#${this.modalOptionSave.id}`).modal('open');  // faz abir o modal com a informação com o id correto
            },
            modalEdit(category, parent){
                this.title = 'Editar categoria'
                this.categorySave = {
                    id: category.id,  // id categoria que estamos a editar
                    name: category.name,
                    parent_id: category.parent_id,
                };
                this.category = category;
                this.parent = parent;
                $(`#${this.modalOptionSave.id}`).modal('open');  // faz abir o modal com a informação com o id correto
            },
            formatCategories(){
                this.categoriesFormatted = CategoryFormat.getCategoriesFormatted(this.categories);
            },
            // fazer reset do nosso scope (categorias)
            resetScope(){
                //reset ao nosso categorySave
                this.categorySave = {
                    id: 0,
                    name: '',
                    parent_id: 0,
                };
                this.category = null;
                this.parent = null;
                this.formatCategories(); // cama novamente o nosso formatCategories para ter uma coleçao de categorias formatada
            },
        },
        events:{
            'category-new'(category){
                this.modalNew(category);
            },
            'category-edit'(category, parent){  // recebemos a categoria e o parent da categori se existir
                this.modalEdit(category, parent);
            },
        }
    }

</script>