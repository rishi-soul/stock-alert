
 const isActive = plan => plan.coverageStatus === CoverageStatus.Active;
            const isMedical = plan => plan.medicalProductPlan;
            const getMember = (plan, relationship, id) =>
                plan.planMembers.find(pm => pm.relationshipToSubscriber === relationship && pm.memberContractId === id);
