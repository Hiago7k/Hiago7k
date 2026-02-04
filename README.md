select distinct (substr(lpad(rrbc.cnpj, 14, 0), 1, 8)) as RAIZ_CNPJ,
               rrb.apelido,
               (select count(1)
                  from cpd.rbx_empresa re
                 where re.apelido = rrb.apelido
                   and re.situacao_cadastral = 'ATIVA'
                 group by re.apelido) as QTD_CNPJs,
               tfdc.lista_projetos,
               rrb.dat_fim,
               decode(rrb.log_sped_fiscal, 0, 'NÃO', 1, 'SIM') AS SPED_LIBERADO,
               (select count(1)
                  from extrator.bx_monitor_cad bmc
                 inner join extrator.bx_monitor_cad_cnpj bmcc
                    on bmcc.num_seq = bmc.num_seq
                 where bmc.funcao = 'BAIXA_ESCRITURACAO_SPED'
                   and bmc.tipo_recorrencia  'U'
                   and bmc.habilitado = 'S'
                   and substr(bmcc.cnpj, 1, 8) =
                       substr(lpad(rrbc.cnpj, 14, 0), 1, 8)
                   and bmcc.habilitado = 'S') AS QTD_TAREFAS,
               (select listagg(bmc.des_tarefa, ', ')
                  from extrator.bx_monitor_cad bmc
                 inner join extrator.bx_monitor_cad_cnpj bmcc
                    on bmcc.num_seq = bmc.num_seq
                 where bmc.funcao = 'BAIXA_ESCRITURACAO_SPED'
                   and bmc.tipo_recorrencia  'U'
                   and bmc.habilitado = 'S'
                   and substr(bmcc.cnpj, 1, 8) =
                       substr(lpad(rrbc.cnpj, 14, 0), 1, 8)
                   and bmcc.habilitado = 'S') AS TAREFAS,
               (select max(bmc.dat_proxima_execucao)
                  from extrator.bx_monitor_cad bmc
                 inner join extrator.bx_monitor_cad_cnpj bmcc
                    on bmcc.num_seq = bmc.num_seq
                 where bmc.funcao = 'BAIXA_ESCRITURACAO_SPED'
                   and bmc.tipo_recorrencia  'U'
                   and bmc.habilitado = 'S'
                   and substr(bmcc.cnpj, 1, 8) =
                       substr(lpad(rrbc.cnpj, 14, 0), 1, 8)
                   and bmcc.habilitado = 'S') AS PROX_EXECUCAO,
              decode(rrb.Log_Efd_Contribuicoes, 0, 'NÃO', 1, 'SIM') AS EFD_LIBERADO,
               (select COUNT(1)
                  from extrator.bx_monitor_cad bmc
                 inner join extrator.bx_monitor_cad_cnpj bmcc
                    on bmcc.num_seq = bmc.num_seq
                 where bmc.funcao = 'BAIXA_ESCRITURACAO_EFD'
                   and bmc.tipo_recorrencia  'U'
                   and bmc.habilitado = 'S'
                   and substr(bmcc.cnpj, 1, 8) =
                       substr(lpad(rrbc.cnpj, 14, 0), 1, 8)
                   and bmcc.habilitado = 'S') AS QTD_TAREFAS,
               (select listagg(bmc.des_tarefa, ', ')
                  from extrator.bx_monitor_cad bmc
                 inner join extrator.bx_monitor_cad_cnpj bmcc
                    on bmcc.num_seq = bmc.num_seq
                 where bmc.funcao = 'BAIXA_ESCRITURACAO_EFD'
                   and bmc.tipo_recorrencia  'U'
                   and bmc.habilitado = 'S'
                   and substr(bmcc.cnpj, 1, 8) =
                       substr(lpad(rrbc.cnpj, 14, 0), 1, 8)
                   and bmcc.habilitado = 'S') AS TAREFAS,
               (select max(bmc.dat_proxima_execucao)
                  from extrator.bx_monitor_cad bmc
                 inner join extrator.bx_monitor_cad_cnpj bmcc
                    on bmcc.num_seq = bmc.num_seq
                 where bmc.funcao = 'BAIXA_ESCRITURACAO_EFD'
                   and bmc.tipo_recorrencia  'U'
                   and bmc.habilitado = 'S'
                   and substr(bmcc.cnpj, 1, 8) =
                       substr(lpad(rrbc.cnpj, 14, 0), 1, 8)
                   and bmcc.habilitado = 'S') AS PROX_EXECUCAO ,
               decode(rrb.log_ecf, 0, 'NÃO', 1, 'SIM') AS ECF_LIBERADO,
               (select listagg(bmc.des_tarefa, ',')
                  from extrator.bx_monitor_cad bmc
                 inner join extrator.bx_monitor_cad_cnpj bmcc
                    on bmcc.num_seq = bmc.num_seq
                 where bmc.funcao = 'BAIXA_ESCRITURACAO_ECF'
                   and bmc.tipo_recorrencia  'U'
                   and bmc.habilitado = 'S'
                   and substr(bmcc.cnpj, 1, 8) =
                       substr(lpad(rrbc.cnpj, 14, 0), 1, 8)
                   and bmcc.habilitado = 'S') AS TAREFAS
from cpd.rbx_receita_bx rrb
inner join cpd.rbx_receita_bx_cnpj rrbc
   on rrbc.id_receita_bx = rrb.id
left join cpd.tb_flow_dw_cliente tfdc
   on tfdc.raiz_cnpj = substr(lpad(rrbc.cnpj, 14, 0), 1, 8)
where rrb.dat_fim = sysdate;
