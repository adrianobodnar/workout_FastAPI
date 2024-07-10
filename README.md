<h1>
    <a href="https://www.dio.me/">
     <img align="center" width="70px" src="https://hermes.digitalinnovation.one/assets/diome/logo-full.svg"></a>
    <span>Python AI Backend Developer  </span>
</h1>

Repositório desenvolvido para fins didáticos, criado para armazenar os projetos desenvolvidos no  bootcamp  **bootcamp Coding The Future Vivo - Python AI Backend Developer** da [Digital Innovation One](https://www.dio.me/).


<img align="center" width="250px" src="https://hermes.dio.me/files/assets/ef695d25-f647-45eb-b1ad-a25c124b28ca.png">




### Quem é o FastAPi?

Framework FastAPI, alta performance, fácil de aprender, fácil de codar, pronto para produção. FastAPI é um moderno e rápido (alta performance) framework web para construção de APIs com Python 3.6 ou superior, baseado nos type hints padrões do Python


## Tecnologias Utilizadas

🏆 Pipenv - Controle de versão

🏆 PostgreSQL - Banco de dados com docker-compose

🏆 SQLAlchemy + Pydantic + Alembic - conexão com banco de dados

🏆 FastAPI - Desenvolver a aplicação

---
## Desafio de Projeto da DIO
https://github.com/digitalinnovationone/workout_api

⚠️ Adicionar query parameters nos endpoints
~~~
-Atleta
  -nome
  -cpf
~~~    
   Foi adicionado no arquivo atleta/controller.py

    @router.get(
            path='/nome={nome}', 
            summary='consultar um atleta pelo nome',
            status_code = status.HTTP_200_OK,
            response_model= AtletaOut,
            ) 

    async def query(nome: str, db_session: DatabaseDependency, cpf: str | None = None) -> AtletaOut:
        atleta: AtletaOut = (
        await db_session.execute(select(AtletaModel).filter_by(nome=nome, cpf=cpf))
            ).scalars().first()
     
        if not atleta:
            raise HTTPException(
                status_code = status.HTTP_404_NOT_FOUND, 
                detail= f'Atleta não encontrado com nome: {nome}'
                )
    
        return atleta



⚠️ Customizar response de retorno de endpoints

~~~
- get all
    -atleta
    -nome
    -centro_treinamento
    -categoria
~~~

 Foi criado o schema personalizado em atletas/schemas.py

     class AtletaResponse(BaseSchema):
        nome: Annotated[str, Field(description='Nome do Atleta', example='Joao', max_length=50)]
        categoria: Annotated[CategoriaIn, Field(description='Categoria do Atleta')]
        centro_treinamento: Annotated[CentroTreinamentoAtleta, Field(description='Centro de treinamento do Atleta')]


 Foi adicionado o endpoint no arquivo atleta/controller.py

     @router.get(
            path='/all_atletas', 
            summary='consulta personalizada todos os atletas',
            status_code = status.HTTP_200_OK,
            response_model= list[AtletaResponse],
            ) 


    async def query(db_session: DatabaseDependency) -> list[AtletaResponse]:
        atletas: list[AtletaResponse] = (await db_session.execute(select(AtletaModel))).scalars().all()

        return [AtletaResponse.model_validate(atleta) for atleta in atletas]


⚠️ Manipular exceção de integridade dos dados em cada módulo/tabela.

~~~
    - qlalchemy.exc.IntegrityError e devolver a seguinte mensagem: “Já existe um atleta cadastrado com o cpf: x”
    - status_code: 303
~~~

     await db_session.commit()
        except IntegrityError:
            raise HTTPException(
                status_code=status.HTTP_303_SEE_OTHER,
                detail=f'Já existe um atleta cadastrado com o cpf: {atleta_in.cpf}'
            )

⚠️ Adicionar paginação utilizando a lib: fastapi-pagination
~~~
-limit e offset
~~~

    #Add pagination with SQLAlchemy
    from fastapi_pagination import LimitOffsetPage, Page
    from fastapi_pagination.ext.sqlalchemy import paginate

    @router.get(
            path='/', 
            summary='consultar todos os atletas',
            status_code = status.HTTP_200_OK,
            response_model= LimitOffsetPage[AtletaOut],
            ) 


    async def query(db_session: DatabaseDependency):
    
        return await paginate(db_session, select(AtletaModel))




