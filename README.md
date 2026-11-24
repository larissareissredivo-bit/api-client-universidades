# api-client-universidades
https://6b04d31edb01cf463607a50c.mockapi.io/universidades/:endpoint
"use client"

import { useState } from "react"
import { Button } from "@/components/ui/button"

const universidadesUrl =
  process.env.NEXT_PUBLIC_API_URL ??
  "https://SEU_CODIGO.mockapi.io/universidades"

type Universidade = {
  id: string
  nome: string
  endereco: string
}

function isUniversidade(value: unknown): value is Universidade {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    typeof value.id === "string" &&
    "nome" in value &&
    typeof value.nome === "string" &&
    "endereco" in value &&
    typeof value.endereco === "string"
  )
}

function isUniversidadeList(value: unknown): value is Universidade[] {
  return Array.isArray(value) && value.every(isUniversidade)
}

export default function Page() {
  const [universidades, setUniversidades] = useState<Universidade[] | null>(
    null,
  )
  const [isLoading, setIsLoading] = useState(false)
  const [isCreating, setIsCreating] = useState(false)
  const [deletingUniversidadeId, setDeletingUniversidadeId] = useState<
    string | null
  >(null)
  const [editingUniversidadeId, setEditingUniversidadeId] = useState<
    string | null
  >(null)
  const [isUpdating, setIsUpdating] = useState(false)

  const [nome, setNome] = useState("")
  const [endereco, setEndereco] = useState("")
  const [editingNome, setEditingNome] = useState("")
  const [editingEndereco, setEditingEndereco] = useState("")

  const [error, setError] = useState<string | null>(null)
  const [success, setSuccess] = useState<string | null>(null)

  async function list() {
    setIsLoading(true)
    setError(null)
    setSuccess(null)

    try {
      const response = await fetch(universidadesUrl)

      if (!response.ok) {
        throw new Error(
          `Falha ao carregar universidades (HTTP ${response.status}).`,
        )
      }

      const data: unknown = await response.json()
      if (!isUniversidadeList(data)) {
        throw new Error(
          "A resposta da API não contém uma lista de universidades válida.",
        )
      }

      setUniversidades(data)
    } catch (error) {
      setError(
        error instanceof Error
          ? error.message
          : "Ocorreu um erro ao carregar as universidades.",
      )
    } finally {
      setIsLoading(false)
    }
  }

  async function createUniversidade(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault()
    setIsCreating(true)
    setError(null)
    setSuccess(null)

    try {
      const response = await fetch(universidadesUrl, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          nome: nome.trim(),
          endereco: endereco.trim(),
        }),
      })

      if (!response.ok) {
        throw new Error(
          `Falha ao cadastrar universidade (HTTP ${response.status}).`,
        )
      }

      const data: unknown = await response.json()
      if (!isUniversidade(data)) {
        throw new Error(
          "A resposta da API não contém uma universidade válida.",
        )
      }

      setUniversidades((currentUniversidades) =>
        currentUniversidades ? [...currentUniversidades, data] : [data],
      )
      setNome("")
      setEndereco("")
      setSuccess("Universidade cadastrada com sucesso.")
    } catch (error) {
      setError(
        error instanceof Error
          ? error.message
          : "Ocorreu um erro ao cadastrar a universidade.",
      )
    } finally {
      setIsCreating(false)
    }
  }

  async function deleteUniversidade(universidade: Universidade) {
    if (
      !window.confirm(`Tem certeza que deseja excluir ${universidade.nome}?`)
    ) {
      return
    }

    setDeletingUniversidadeId(universidade.id)
    setError(null)
    setSuccess(null)

    try {
      const response = await fetch(
        `${universidadesUrl}/${encodeURIComponent(universidade.id)}`,
        { method: "DELETE" },
      )

      if (!response.ok) {
        throw new Error(
          `Falha ao excluir universidade (HTTP ${response.status}).`,
        )
      }

      setUniversidades((currentUniversidades) =>
        currentUniversidades
          ? currentUniversidades.filter(
              (currentUniversidade) =>
                currentUniversidade.id !== universidade.id,
            )
          : currentUniversidades,
      )
      setSuccess("Universidade excluída com sucesso.")
    } catch (error) {
      setError(
        error instanceof Error
          ? error.message
          : "Ocorreu um erro ao excluir a universidade.",
      )
    } finally {
      setDeletingUniversidadeId(null)
    }
  }

  function startEditing(universidade: Universidade) {
    setEditingUniversidadeId(universidade.id)
    setEditingNome(universidade.nome)
    setEditingEndereco(universidade.endereco)
    setError(null)
    setSuccess(null)
  }

  function cancelEditing() {
    setEditingUniversidadeId(null)
    setEditingNome("")
    setEditingEndereco("")
  }

  async function updateUniversidade(
    event: React.FormEvent<HTMLFormElement>,
    universidade: Universidade,
  ) {
    event.preventDefault()
    setIsUpdating(true)
    setError(null)
    setSuccess(null)

    try {
      const response = await fetch(
        `${universidadesUrl}/${encodeURIComponent(universidade.id)}`,
        {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            nome: editingNome.trim(),
            endereco: editingEndereco.trim(),
          }),
        },
      )

      if (!response.ok) {
        throw new Error(
          `Falha ao atualizar universidade (HTTP ${response.status}).`,
        )
      }

      const data: unknown = await response.json()
      if (!isUniversidade(data) || data.id !== universidade.id) {
        throw new Error(
          "A resposta da API não contém uma universidade válida.",
        )
      }

      setUniversidades(
        (currentUniversidades) =>
          currentUniversidades?.map((currentUniversidade) =>
            currentUniversidade.id === universidade.id
              ? data
              : currentUniversidade,
          ) ?? currentUniversidades,
      )
      cancelEditing()
      setSuccess("Universidade atualizada com sucesso.")
    } catch (error) {
      setError(
        error instanceof Error
          ? error.message
          : "Ocorreu um erro ao atualizar a universidade.",
      )
    } finally {
      setIsUpdating(false)
    }
  }

  return (
    <main className="mx-auto min-h-screen max-w-4xl space-y-8 px-6 py-12">
      <section className="space-y-5 rounded-xl border bg-card p-6 shadow-sm">
        <div>
          <h1 className="text-2xl font-semibold">Cadastrar universidade</h1>
          <p className="mt-1 text-sm text-muted-foreground">
            Preencha os dados para adicionar uma nova universidade.
          </p>
        </div>
        <form className="space-y-4" onSubmit={createUniversidade}>
          <div className="space-y-2">
            <label htmlFor="universidade-nome" className="text-sm font-medium">
              Nome
            </label>
            <input
              id="universidade-nome"
              name="nome"
              required
              value={nome}
              onChange={(event) => setNome(event.target.value)}
              className="h-10 w-full rounded-lg border bg-background px-3 text-sm outline-none focus-visible:ring-2 focus-visible:ring-ring"
            />
          </div>
          <div className="space-y-2">
            <label
              htmlFor="universidade-endereco"
              className="text-sm font-medium"
            >
              Endereço
            </label>
            <input
              id="universidade-endereco"
              name="endereco"
              required
              value={endereco}
              onChange={(event) => setEndereco(event.target.value)}
              className="h-10 w-full rounded-lg border bg-background px-3 text-sm outline-none focus-visible:ring-2 focus-visible:ring-ring"
            />
          </div>
          <Button type="submit" disabled={isCreating}>
            {isCreating ? "Cadastrando..." : "Cadastrar universidade"}
          </Button>
        </form>
        {error && (
          <p role="alert" className="text-sm text-destructive">
            {error}
          </p>
        )}
        {success && (
          <p role="status" className="text-sm text-primary">
            {success}
          </p>
        )}
      </section>

      <section className="space-y-4">
        <div className="flex items-center justify-between gap-4">
          <h2 className="text-xl font-semibold">Universidades</h2>
          <Button
            onClick={list}
            disabled={
              isLoading ||
              editingUniversidadeId !== null ||
              deletingUniversidadeId !== null
            }
            variant="outline"
          >
            {isLoading ? "Carregando..." : "Listar universidades"}
          </Button>
        </div>
        {universidades && (
          <div className="overflow-x-auto rounded-lg border">
            <table className="w-full border-collapse text-sm">
              <thead className="bg-muted">
                <tr>
                  <th className="px-4 py-3 text-left font-medium">ID</th>
                  <th className="px-4 py-3 text-left font-medium">Nome</th>
                  <th className="px-4 py-3 text-left font-medium">Endereço</th>
                  <th className="px-4 py-3 text-right font-medium">Ações</th>
                </tr>
              </thead>
              <tbody>
                {universidades.map((universidade) => (
                  <tr key={universidade.id} className="border-t">
                    <td className="px-4 py-3 font-mono text-xs text-muted-foreground">
                      {universidade.id}
                    </td>
                    <td className="px-4 py-3">
                      {editingUniversidadeId === universidade.id ? (
                        <>
                          <label
                            htmlFor={`universidade-nome-${universidade.id}`}
                            className="sr-only"
                          >
                            Nome de {universidade.nome}
                          </label>
                          <input
                            id={`universidade-nome-${universidade.id}`}
                            form={`universidade-edit-${universidade.id}`}
                            required
                            value={editingNome}
                            onChange={(event) =>
                              setEditingNome(event.target.value)
                            }
                            className="h-9 w-full min-w-32 rounded-lg border bg-background px-2 text-sm outline-none focus-visible:ring-2 focus-visible:ring-ring"
                          />
                        </>
                      ) : (
                        universidade.nome
                      )}
                    </td>
                    <td className="px-4 py-3">
                      {editingUniversidadeId === universidade.id ? (
                        <>
                          <label
                            htmlFor={`universidade-endereco-${universidade.id}`}
                            className="sr-only"
                          >
                            Endereço de {universidade.nome}
                          </label>
                          <input
                            id={`universidade-endereco-${universidade.id}`}
                            form={`universidade-edit-${universidade.id}`}
                            required
                            value={editingEndereco}
                            onChange={(event) =>
                              setEditingEndereco(event.target.value)
                            }
                            className="h-9 w-full min-w-40 rounded-lg border bg-background px-2 text-sm outline-none focus-visible:ring-2 focus-visible:ring-ring"
                          />
                        </>
                      ) : (
                        universidade.endereco
                      )}
                    </td>
                    <td className="px-4 py-3 text-right">
                      {editingUniversidadeId === universidade.id ? (
                        <form
                          id={`universidade-edit-${universidade.id}`}
                          onSubmit={(event) =>
                            updateUniversidade(event, universidade)
                          }
                          className="flex justify-end gap-2"
                        >
                          <Button type="submit" disabled={isUpdating}>
                            {isUpdating ? "Salvando..." : "Salvar"}
                          </Button>
                          <Button
                            type="button"
                            variant="outline"
                            disabled={isUpdating}
                            onClick={cancelEditing}
                          >
                            Cancelar
                          </Button>
                        </form>
                      ) : (
                        <div className="flex justify-end gap-2">
                          <Button
                            type="button"
                            variant="outline"
                            disabled={
                              editingUniversidadeId !== null ||
                              deletingUniversidadeId !== null
                            }
                            onClick={() => startEditing(universidade)}
                            aria-label={`Editar ${universidade.nome}`}
                          >
                            Editar
                          </Button>
                          <Button
                            type="button"
                            variant="destructive"
                            disabled={
                              deletingUniversidadeId !== null ||
                              editingUniversidadeId !== null
                            }
                            onClick={() => deleteUniversidade(universidade)}
                            aria-label={`Excluir ${universidade.nome}`}
                          >
                            {deletingUniversidadeId === universidade.id
                              ? "Excluindo..."
                              : "Excluir"}
                          </Button>
                        </div>
                      )}
                    </td>
                  </tr>
                ))}
                {universidades.length === 0 && (
                  <tr>
                    <td
                      colSpan={4}
                      className="px-4 py-6 text-center text-muted-foreground"
                    >
                      Nenhuma universidade cadastrada.
                    </td>
                  </tr>
                )}
              </tbody>
            </table>
          </div>
        )}
      </section>
    </main>
  )
}
