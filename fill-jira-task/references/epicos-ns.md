# Example area epics (NS)

This mapping is an example of choosing a parent epic by system area for one project. It is not a catalog for other projects. Do not copy these keys onto an issue in another project.

When this list may be outdated, or the issue belongs to another project, list that project's open epics before choosing. Keep the issue-type clause:

```text
project = <project> AND issuetype = Epic
```

Adapt the project key and the issue-type name or id to the actual site. Prefer an unresolved epic when the connection can express that without dropping `issuetype = Epic`. Read each candidate's summary, description, and status. A shared domain word is not a match.

| Key | Area | What the area covers |
| --- | --- | --- |
| NS-880 | Agenda e Agendamentos | Scheduling and appointments. |
| NS-881 | Evoluções | Performing a clinical attendance or session (realizar um atendimento/sessão clínica). Not a generic bucket for other clinical work. |
| NS-882 | Painel de Gestão | Management panel. |
| NS-883 | Pacientes | Patients. |
| NS-886 | Plataforma | Platform. |
| NS-887 | Programas | Programs. |
| NS-889 | Usuários | Users. |

The parent epic is the area the task touches. Do not select a generic bucket, and do not select a domain concept that is not that area. Never create an epic to fill a gap; report the missing area instead. Hierarchy, ambiguity, and write rules stay in the [skill entrypoint](../SKILL.md).
